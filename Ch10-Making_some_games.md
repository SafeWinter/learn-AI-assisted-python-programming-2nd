# Ch10: Making some games



> **本章概要**
>
> - 在程序中引入随机性；
> - 猜数游戏的设计与实现；
> - 掷骰游戏的设计与实现。

本章聚焦 AI 辅助编程在小型游戏中的落地应用。从随机性的使用入手，分别演示了两个小游戏的设计过程和 AI 辅助实现过程，旨在进一步强化问题分解能力和自顶向下的设计思维的培养。



## 10.1 关于游戏编程

本章重点介绍 2D 桌游。这类游戏应用通常包括两个阶段：

- 游戏准备阶段：对应 **设置函数（setup function）**，负责游戏的初始化；
- 游戏运行阶段：对应 **更新函数（update function）**，负责根据玩家操作或时间流逝来改变游戏状态；

以棋牌类游戏为例，通常由玩家轮流操作，若无人胜出则视为平局并重新开始。每次操作都会设计游戏状态的变更（棋盘布局、玩家资源等）。

基本流程如图所示：

![](assets/10.2.png)

**图 10.1 大多数电子游戏的基本流程**

入门游戏编程，推荐使用 `Python` 的 `pygame` 模块；深入钻研游戏设计可尝试 `Unity` 搞 3D 游戏开发。本章只基于 `Python` 基础语法简要演示 AI 编程在游戏开发中的具体应用，不涉及上述工具。



## 10.2 随机性的引入

通过 `Python` 内置模块 `random` 实现：

```python
>>> import random
>>> random.randint(0, 9)
5 # 值域：[0, 9)
>>> random.random()
0.27305746549408105 # 值域：[0, 1)
```

典型场景：

- 对方操作（骰子数、前进步数、攻击造成的伤害值等）；
- 谜题初始化布局（扫雷雷区分布等）；
- 表示事件概率（通常以浮点数形式）。



## 10.3 实战1：Bulls & Cows

### 10.3.1 玩法说明

类似 `Wordle` 词云游戏——

- 开局后，系统会生成一个无重复数字的四位数作为谜底；
- 玩家通过系统提示的命中和半命中数据反复尝试不同的数字组合，直到完全猜对为止（胜出）；
- 为了适当加大难度，本例还有 **最大尝试次数** 的限制：超过最大次数仍未能猜中谜底则宣布失败，游戏结束。

所谓的命中，就是指数字和所在的数位均和谜底一致的情况，其数量用 `Bulls` 表示；数字正确但数位错误，则为半命中，用 `Cows` 表示。

对 `Wordle` 不熟的朋友可以访问书中提供的 [免费网页版](https://www.mathsisfun.com/games/bulls-and-cows.html) 试玩：

![](assets/10.3.png)

**图 10.2 Bulls and Cows 线上免费版截图**



### 10.3.2 自上而下的设计思路

先用伪代码设计游戏流程：

```markdown
游戏设置阶段：
    随机生成谜底（secret_code）

游戏运行阶段：
while 尚未胜出 and 还有尝试次数:
    提示玩家输入猜测值（guess）
    读取有效的猜测值
    对比 gress 与 secret_code：
    if gress == secret_code:
        游戏胜出，提示玩家
    else
        给出 guess 的对比结果
    刷新总猜测次数

若玩家剩余次数为 0，则公布谜底，游戏结束。
```

根据伪代码，假设最顶层函数的名称为 `play`，则可以分为以下三个子函数：

1. `random_string`：用于系统生成四位字符串谜底，并确保每位数字不重复。谜底的数据类型之所以不能为整型数字，主要是不支持首位为 `0` 的情况，其次在提取每位数字时也不如字符串方便。
2. `get_guess`：用于生成玩家猜测结果，其中涉及一系列校验规则（位数也为 `4`、纯数字、无重复等），因此用该函数单独处理。
3. `guess_result`：有了谜底和猜测值，就可以计算对比结果了，即分别统计 `Bulls` 和 `Cows` 的值。这一步也较为繁琐，也放到子函数单独处理。

由于胜出条件就是简单的字符串比较，因此无需子函数；而尝试次数（假设为 10 次）的迭代和更新可以换成 `for` 循环，这样就不用手动维护剩余次数了，并且一旦遍历完所有次数都没猜对，则直接公布答案，宣布游戏结束。

这样 `play` 函数与三个子函数就可以确立为如下结构：

```mermaid
graph LR
    a[play] --> b[random_string]
    a --> c[get_guess]
    a --> d[guess_result]
```

上述分析过程出现了两处硬编码：最大尝试次数 10 和谜底（或猜测值）的总位数 4。这在程序设计中叫做 **魔数（magic number）**。设计应用时应尽量避免出现魔数，可以改为从 `play` 函数传参：

```python
def play(num_digits=4, num_guesses=10):
```

运行过程中，参数 `num_guesses` 用于控制最外层 `for` 循环；而参数 `num_digits` 则可以分别传入前两个子函数，于是有了如下签名：

```python
def random_string(length=4):
    # snip
    
def get_guess(length=4):
    # snip
```

而最后的 `guess_result` 的参数列表就是以上两个子函数的返回值：

```python
def guess_result(guess, secret_code):
```

至此，问题分解环节就全部结束了，接下来是自底向上实现各个函数，用到的方法也是 `GitHub Copilot` 的实时补全功能，依据是紧跟在每个函数签名后的 `docstring` 描述。



### 10.3.3 自底向上的函数实现过程

根据上一节的问题拆解，先从子函数开始补全具体逻辑，再组合到顶层的 `play` 函数中。

#### 1 补全 random_string 函数

这里沿用第 7 章介绍的 `docstring` 的用法，先补全 `random_string` 的功能描述：

```python
def random_string(length=4):
    '''
    length is an integer
    
    Return a string of the given length, where each character 
    is a digit from 0 to 9, and with no repeated digits.
    '''
```

通过回车换行，`Copilot` 会逐行补全代码实现。经过一年多的打磨，`AI` 助手直接给出了最优方案：

```python
def random_string(length=4):
    # -- snip --
    digits = list('0123456789')
    random.shuffle(digits)
    return ''.join(digits[:length])
```



#### 2 补全 get_guess 函数

同理可得 `get_guess` 的功能描述内容，先说明参数列表，再给出整体的功能描述：

```python
def get_guess(length=4):
    '''
    length is an integer.
 
    Keep asking the player to enter a string where each character 
    is a digit from 0 to 9, until they enter a valid guess.
    A valid guess has the given length and has no repeated digits.
    '''
```

可能是吸收了第一版的训练数据，`Copilot` 直接给出了带校验提示信息的最佳版本，可读性大幅增强了：

```python
def get_guess(length=4):
    # -- snip --
    while True:
        guess = input(f'Enter a {length}-digit guess with no repeated digits: ')
        if len(guess) != length:
            print(f'Your guess must be {length} digits long.')
            continue
        if not guess.isdigit():
            print('Your guess must only contain digits from 0 to 9.')
            continue
        if len(set(guess)) != length:
            print('Your guess must not have repeated digits.')
            continue
        return guess
```



#### 3 补全 guess_result 函数

比较意外的是 `guess_result` 的补全结果，先按书中提示添加功能描述。为了增强代码的健壮性，还追加了两个测试用例：

```python
def guess_result(guess, secret_code):
    '''
    guess and secret_code are strings of the same length.
 
    Return a list of two values:
    the first value is the number of indices in guess where
    the character at that index matches the character at the
    same index in secret_code; the second value is the 
    number of indices in guess where the character at that 
    index exists at a different index in secret_code.
    
    >>> guess_result('3821', '1862')
    [1, 2]
    >>> guess_result('1234', '4321')
    [0, 4]
    '''
```

首次补全时，`Copilot` 给出了看似简洁的版本：

```python
def guess_result(guess, secret_code):
    # -- snip --
    correct_position = sum(g == s for g, s in zip(guess, secret_code))
    correct_digit = sum(min(guess.count(d), secret_code.count(d)) for d in set(guess)) - correct_position
    return [correct_position, correct_digit]
```

上述代码灵活运用了 `Python` 的列表展开式写法，统计命中数还算好懂，但半命中数的思路就有点过于跳跃了。为此，可以让 `Copilot` 给出解释：

> **我**：解释一下第52行（即上述代码第 4 行）
>
> **Copilot**：第52行是这一行：
>
> ```python
> correct_digit = sum(min(guess.count(d), secret_code.count(d)) for d in set(guess)) - correct_position
> ```
>
> 解释（简短）：
>
> - 先对 `guess` 中每个不同的数字 d（`set(guess)`）计算两者出现次数的较小值：`min(guess.count(d), secret_code.count(d))`，这给出“在两者中都出现的该数字的最大配对数”（处理重复数字）。
> - 对所有不同数字求和，得到“位置不考虑时共有多少个匹配的数字”。
> - 再减去 `correct_position`（前一行计算的完全位置匹配数），得到“存在于 `secret_code` 但位置不同的数字数目”。
>
> 举例：`guess='3821', secret='1862'`：
>
> - 各数字 `min` 总和 `= 3`（数字 1、2、8 三个在两边都出现）；
> - `correct_position = 1`（数字 8 在同一位置）；
> - `correct_digit = 3 - 1 = 2`（两位数字存在但位置不同）。
>
> 注意：这种写法清晰且能处理重复数字，但每次 `count` 是线性扫描，复杂度较高（字符串较长或大量调用时可优化）。

注意到最后一句，每次 `count` 会由于线性扫描产生性能问题，为什么旧版演示中没有这样写呢？仔细检查我的函数文档，原来 `Copilot` 没有理解 `guess_result` 的两个参数值都是做了去重处理的，因此用 `min` 函数 + `count` 函数来统计 `Cows` 的值。这再一次说明 `Copilot` 即便在同一语境下，仍然有可能丢失上下文中的关键信息，给出的代码实现只局限于当前函数，无法从全局理解该函数的实际作用（莫非是 `Copilot` 免费模型的锅？）。

找到原因就好办了，要么修改 `docstring`，要么直接在 `Copilot Chat` 窗口让它重写，于是有了以下改进版：

```python
def guess_result(guess, secret_code):
    # -- snip --
    # 第一步：位置完全相同的数字个数
    correct_position = sum(g == s for g, s in zip(guess, secret_code))
    
    # 第二步：数字存在但位置不同的个数
    # 由于 guess 和 secret_code 中都没有重复数字，
    # 只需检查 guess 中的每个数字是否在 secret_code 中出现过
    # 再排除已经在正确位置的数字
    correct_digit = sum(g in secret_code and g != s for g, s in zip(guess, secret_code))
    
    return [correct_position, correct_digit]
```

虽然没了 `min` 函数，但还是差点意思：明明是同一套遍历，却要重复运行一次来统计第二个指标，确实算不上智能。只能继续提示 `Copilot` 完善：

> **我**：还有点小问题，新版实现用了两次同样的 zip 遍历，其实两种情况可以放到一次遍历中进行
>
> **Copilot**：你说得对！可以在一次遍历中同时计算两个值。这样更高效：
>
> ```python
> def guess_result(guess, secret_code):
>     # -- snip --
>     correct_position = 0
>     correct_digit = 0
>     
>     for g, s in zip(guess, secret_code):
>         if g == s:
>             correct_position += 1
>         elif g in secret_code:
>             correct_digit += 1
>     
>     return [correct_position, correct_digit]
> ```
>
> 优化点：
>
> - 只遍历一次 `zip(guess, secret_code)`。
> - 在单次遍历中同时判断：位置是否相同，或数字是否存在但位置不同。
> - 时间复杂度：`O(n)`（n 为字符串长度）。
> - 代码逻辑清晰易懂。

到这里 `Copilot` 才给出书中的最佳实践版本。这说明 `LLM` 补全的内容还是离不开真人开发者的二次审核。信任依旧无法替代监督。



#### 4 补全 play 函数

补全所有子函数后，该轮到最顶层的 `play` 函数了。先添加函数 `docstring`：

```python
def play(num_digits=4, num_guesses=10):
    '''
    Generate a random string with num_digits digits.
    The player has num_guesses guesses to guess the random 
    string. After each guess, the player is told how many 
    digits in the guess are in the correct place, and how 
    many digits exist but are in the wrong place.
    '''
```

有了 `guess_result` 的小插曲，`play` 函数的实现非常顺利，直接给出“标准答案”了：

```python
def play(num_digits=4, num_guesses=10):
    # -- snip --
    secret_code = random_string(num_digits)
    print(f'I have generated a {num_digits}-digit secret code with no repeated digits.')
    print(f'You have {num_guesses} guesses to find the code.')
    for guess_num in range(num_guesses):
        print(f'Guess #{guess_num + 1}:')
        guess = get_guess(num_digits)
        correct_position, correct_digit = guess_result(guess, secret_code)
        print(f'Correct: {correct_position}, Misplaced: {correct_digit}')
        if correct_position == num_digits:
            print('Congratulations! You guessed the secret code!')
            return
            
    print(f'Sorry, you have used all your guesses. The secret code was {secret_code}.')
```



### 10.3.4 实测与完善

实测游戏前，先补全 `play` 函数的自动调用语句：

```python
if __name__ == '__main__':
    play()
```

然后在命令行中执行 `py demo.py`，实测过程如下：

![](assets/10.4.png)

**图 10.3 本地实测 Bulls and Cows 猜数游戏命令行版本**



纯命令行界面还是有点简陋，让 `Copilot` 提供一个 `GUI` 界面：

> **我**：你能将这个程序改为使用图形界面而不是文本界面吗？
>
> **Copilot**：下面把原来的命令行 `play` 替换为一个简单的 `tkinter` 图形界面实现；保留了 `random_string` 与 `guess_result` 的逻辑。将下面代码保存到 `f:\mydesktop\ai-demo\demo.py`（覆盖原文件或替换文件尾部的 `play` / **main** 部分）。
>
> （略）

由于 `GUI` 界面设计事件驱动编程，不适合作为 `Python` 入门知识演示，作者也没有深入考察，只是提供了一个参考资料[^1]。感兴趣的朋友可以自行深挖。

这里直接给出 `VSCode GitHub Copilot` 默认模型给出的 `GUI` 版实现：

```python
import random
# 替换命令行 play 为 tkinter GUI 实现
import tkinter as tk
from tkinter import messagebox

def random_string(length=4):
    '''
    length is an integer
    
    Return a string of the given length, where each character 
    is a digit from 0 to 9, and with no repeated digits.
    '''
    digits = list('0123456789')
    random.shuffle(digits)
    return ''.join(digits[:length])
    
def guess_result(guess, secret_code):
    '''
    guess and secret_code are strings of the same length.
 
    Return a list of two values:
    the first value is the number of indices in guess where
    the character at that index matches the character at the
    same index in secret_code; the second value is the 
    number of indices in guess where the character at that 
    index exists at a different index in secret_code.
    
    >>> guess_result('3821', '1862')
    [1, 2]
    >>> guess_result('1234', '4321')
    [0, 4]
    '''
    correct_position = 0
    correct_digit = 0
    
    for g, s in zip(guess, secret_code):
        if g == s:
            correct_position += 1
        elif g in secret_code:
            correct_digit += 1
    
    return [correct_position, correct_digit]


class CodeBreakerGUI:
    def __init__(self, master, num_digits=4, num_guesses=10):
        self.master = master
        self.num_digits = num_digits
        self.num_guesses = num_guesses
        self.secret_code = ''
        self.attempts = 0

        master.title('Bulls & Cows - Code Breaker Game')

        self.info_label = tk.Label(master, text='')
        self.info_label.pack(padx=10, pady=6)

        self.attempts_label = tk.Label(master, text='')
        self.attempts_label.pack(padx=10)

        entry_frame = tk.Frame(master)
        entry_frame.pack(pady=6)
        self.guess_entry = tk.Entry(entry_frame, width=20)
        self.guess_entry.pack(side=tk.LEFT, padx=(0,6))
        self.submit_btn = tk.Button(entry_frame, text='Submit', command=self.submit_guess)
        self.submit_btn.pack(side=tk.LEFT)

        self.history_text = tk.Text(master, width=40, height=10, state='disabled')
        self.history_text.pack(padx=10, pady=6)

        control_frame = tk.Frame(master)
        control_frame.pack(pady=6)
        self.new_btn = tk.Button(control_frame, text='New Game', command=self.new_game)
        self.new_btn.pack(side=tk.LEFT, padx=6)
        self.quit_btn = tk.Button(control_frame, text='Quit', command=master.quit)
        self.quit_btn.pack(side=tk.LEFT, padx=6)

        self.new_game()

    def new_game(self):
        self.secret_code = random_string(self.num_digits)
        self.attempts = 0
        self.info_label.config(text=f'I have generated a {self.num_digits}-digit secret code with no repeated digits.')
        self.update_attempts()
        self.clear_history()
        self.guess_entry.delete(0, tk.END)
        self.guess_entry.focus_set()
        self.submit_btn.config(state='normal')

    def update_attempts(self):
        remaining = self.num_guesses - self.attempts
        self.attempts_label.config(text=f'Guesses left: {remaining} / {self.num_guesses}')

    def append_history(self, text):
        self.history_text.config(state='normal')
        self.history_text.insert(tk.END, text + '\n')
        self.history_text.see(tk.END)
        self.history_text.config(state='disabled')

    def clear_history(self):
        self.history_text.config(state='normal')
        self.history_text.delete('1.0', tk.END)
        self.history_text.config(state='disabled')

    def validate_guess(self, guess):
        if len(guess) != self.num_digits:
            messagebox.showwarning('Invalid guess', f'Guess must be {self.num_digits} digits long.')
            return False
        if not guess.isdigit():
            messagebox.showwarning('Invalid guess', 'Guess must contain only digits 0-9.')
            return False
        if len(set(guess)) != self.num_digits:
            messagebox.showwarning('Invalid guess', 'Digits must not repeat.')
            return False
        return True

    def submit_guess(self):
        guess = self.guess_entry.get().strip()
        if not self.validate_guess(guess):
            return
        self.attempts += 1
        correct_position, correct_digit = guess_result(guess, self.secret_code)
        self.append_history(f'#{self.attempts}  {guess}  -> Correct: {correct_position}, Misplaced: {correct_digit}')
        self.update_attempts()
        self.guess_entry.delete(0, tk.END)
        if correct_position == self.num_digits:
            messagebox.showinfo('Victory', f'Congratulations! You guessed the secret code {self.secret_code} in {self.attempts} guesses.')
            self.submit_btn.config(state='disabled')
        elif self.attempts >= self.num_guesses:
            messagebox.showinfo('Game Over', f'Sorry, you used all guesses. Secret code was: {self.secret_code}')
            self.submit_btn.config(state='disabled')

if __name__ == '__main__':
    root = tk.Tk()
    app = CodeBreakerGUI(root, num_digits=4, num_guesses=10)
    root.mainloop()
```

实测效果如下（明显更智能）：

![](./assets/10.1.png)

**图 10.4 由 Copilot Chat 提供的 GUI 版 Bulls and Cows 游戏实测界面**



## 10.4 实战2：Bogart

### 10.4.1 游戏规则

本章要演示开发的第二个小游戏名叫 `Bogart`，是一款根据同名游戏改编简化后的双人掷骰子游戏。原版游戏最初是为华盛顿州西雅图的 **Dalmuti's Restaurant** 餐馆设计的，后来经过修订更名为 `Crash` 出售。由于这也是本书上一版的最后一个实战案例，难度也是压轴级的，看看它的游戏规则就知道了——

开局前，奖池内先由系统放入 1 枚筹码（chip），随机选取一方掷骰子。若点数为 1 则出局，直接换人；否则由玩家决定是继续：

- 如果放弃，则拿走奖池内的所有筹码，换另一玩家掷骰子（系统又从放 1 枚筹码、掷 1 个骰子开始）；
- 如果继续，则系统将向奖池内再投放 2 枚筹码，同时骰子数也变为 2，然后再投掷；

投掷过程中，只要出现一次 1 点，就直接出局换人；否则再次决定是否继续：放弃则拿走奖池筹码并换人；继续则系统再投放 3 枚筹码，骰子也变为 3 个……以此类推，直到任一玩家获得的筹码超过 30 枚、或者骰子已经追加到 5 个、且 5 个全不为 1 点的一方，为最终获胜方，游戏方才结束。



### 10.4.2 游戏流程拆解

要开发这样一款游戏，最核心的任务在于梳理出它的游戏流程图（这一步作者完全让开发者解决，不借助 AI）：

![](assets/10.5.png)

**图 10.5 简化版 Bogart 游戏中一回合的游戏流程梳理图**

该游戏原始版本的说明我也看了，有必要补充两个细节：

- 原本是设计的 2 至 6 人同时参与，随机选一人开始后沿顺时针切换玩家，本例简化为两人互换；
- 原版每次追加筹码都是从一个筹码库（**Bank**）直接追加，而不是让玩家自掏腰包。



### 10.4.3 自顶向下的流程设计

对照上面梳理的游戏流程，就可以大体梳理出如下的执行顺序：

1. 初始化奖池，并让 **玩家1** 和 **玩家2** 的筹码归零（游戏设置阶段的任务）；
2. 随机选择 **玩家1** 或 **玩家2** 开始游戏（游戏设置阶段的任务）；
3. 进入游戏运行阶段：在游戏结束前，循环执行下列操作：
   1. 打印奖池筹码数、**玩家1** 筹码数、以及 **玩家2** 的筹码数。
   2. 当前玩家体验一轮完整回合（投掷、决定）。
   3. 若当前玩家决定拿钱走人，则拿走奖池筹码，奖池筹码数归零。
   4. 换另一位玩家开始游戏。
4. 游戏结束：打印赢得比赛的玩家名称（**玩家1** 或 **玩家2**）。

仔细观察上述流程，看看哪些节点可以直接编程，哪些需要抽取子函数独立完成：

步骤 `1` 和 `2` 都是简单的赋值，无需子函数；

步骤 `3` 需要判定游戏是否结束，分两种情况：任一玩家的筹码数是否达到 30 枚，或者骰子数是否为 5。这里可以引入子函数，名称就叫 `game_over`，参数有三个：**玩家1** 的筹码、**玩家2** 的筹码、上一轮的骰子组合；

步骤 `3.1` 为简单输出，无需子函数；

步骤 `3.2`、`3.3`、`3.4` 均为核心逻辑，先跳过本轮分析；

步骤 `4` 就是简单的打印逻辑，无需子函数。

于是剩下这三个核心步骤：

2. 当前玩家体验一次完整回合。
3. 若当前玩家决定拿钱走人，则拿走奖池筹码，奖池筹码数归零。
4. 换另一位玩家开始游戏。

这三个步骤中，`3.2` 又是最核心的一步。其实原书这一段漏了一个关键细节：为什么要分成这三步进行？按游戏规则，从当前玩家开始掷骰子，到最终换人前，受影响的变量共有四个：奖池筹码数、**玩家1**、**玩家2** 的筹码数、以及掷骰子的结果。如果就设计成一个函数，那么参数和返回值都要保持四个，变量太多，不符合 `Python` 编程的最佳实践。再仔细观察，其实可以剔除两个玩家的筹码数，仅将另两个从子函数中更新后返回给主流程即可，然后再在主流程中根据当前玩家是谁来更新对应的筹码。这样一来既控制了子函数的复杂度，又能减少次要变量的反复传参、更新和返回，因此便有了 `3.2` 和 `3.3` 两个子步骤。

于是，`3.2` 中的完整回合就是一个独立的子函数，不妨命名为  `take_full_turn`，接收开始本轮回合的奖池筹码数 `pot_chains` 为参数，返回更新后的 `pot_chains` 和最终的投掷结果 `rolls`；而在 `take_full_turn` 内部，主要完成两个任务：一是生成骰子序列，参数是具体的骰子数，不妨叫 `roll_dice`，投出结果后，根据玩家是否继续，来决定下一次的奖池数和骰子数（分别在本次基础上加 1），直到玩家出局或主动退出该回合。而循环投掷的判定条件为点数不为 1 且玩家选择继续，可以单列一个子函数 `turn_over`，表示本轮掷骰子是否继续。

而步骤 `3.3` 则根据最新的奖池筹码和投掷结果，决定是否可以给指定玩家累加筹码，如果可以，则说明没有出局，并且累加后的奖池筹码还须清零；但如果是出局，则不累加，奖池的筹码滚入下一个回合。这一步并不复杂，因此无需创建子函数。

最后，步骤 `3.4` 虽然表面看只是换下一个玩家，其实还暗藏了一个前提：游戏还没有结束。如果已经结束了，这一步也是可以跳过的，因此可以复用 `game_over` 函数，然后再更新当前玩家 `current_player`。这一步可以抽取一个子函数 `switch_player`，参数即 `game_over` 需要的参数，外加一个 `current_player`。

于是 `3.2` 的大致结构就出来了：其中肯定包含一个 `while` 循环，判定条件为投掷结果是否出局；在循环体中，如果选择继续，则迭代更新投掷结果和奖池筹码；否则直接跳出循环。一旦跳出循环，则表示该玩家本轮回合结束，此时的投掷结果和奖池筹码将作为后续步骤的参数，完成玩家筹码和奖池状态的更新。

另外，不管投几个骰子，逻辑都是类似的，都是从 `random` 模块逐个产生，最终形成一个随机序列，因此可以通过一个新的子函数专门处理。

最终梳理出的外层函数 `play` 的子函数如下：

![](assets/10.6.png)

**图 10.6 简化版 Bogart 游戏的函数拆解示意图**



### 10.4.4 自底向上的函数实现

梳理完每层的函数后，再从叶子函数开始往上实现各业务逻辑。

#### 1 game_over 函数

首先是公用的 `game_over`，参数有三个：**玩家1** 的筹码数 `player1`、**玩家2** 的筹码数 `player2`，以及当前的骰子结果 `rolls`。手动补全函数签名、各参数含义及函数功能特性描述（`docstring`）如下：

```python
def game_over(player1, player2, rolls):
    '''
    player1 is the number of chips that player 1 has.
    player2 is the number of chips that player 2 has.
    rolls is the last list of dice rolls.
 
    Return True if the game is over, False otherwise.
    
    The game is over if player1 has at least 30 chips, 
    or player 2 has at least 30 chips, 
    or there are 5 rolls none of which is a 1.
    '''
    if player1 >= 30 or player2 >= 30:
        return True
    if len(rolls) == 5 and 1 not in rolls:
        return True
    return False
```

这里其实可以联立三个判定条件，改为：

```python
def game_over(player1, player2, rolls):
    # -- snip --
    return player1 >= 30 or player2 >= 30 or \
            (len(rolls) == 5 and not 1 in rolls) 
```



#### 2 switch_player 函数

有了 `game_over` 判定函数，就可以先实现依赖它的 `switch_player` 函数了。参数只需要新增一个 `current_player` 即可，最终返回切换后的 `current_player`：

```python
def switch_player(player1, player2, rolls, current_player):
    '''
    player1 is the number of chips that player 1 has.
    player2 is the number of chips that player 2 has.
    rolls is the last list of dice rolls.
    current_player is the current player (1 or 2).
    
    If the game is not over, switch current_player to the other player.
    Return the new current_player.
    '''
    if not game_over(player1, player2, rolls):
        current_player = 1 if current_player == 2 else 2
    return current_player
```

总进度就完成了三个节点（和原书略有不同）：

![](assets/10.7.png)



#### 3 roll_dice 函数

该函数根据传入的骰子个数，生成长度为该个数的随机骰子结果：

```python
import random
def roll_dice(n):
    '''
    Create a list of n random integers between 1 and 6.
    Print each of these integers, and return the list.
    '''
    return list(random.randint(1, 6) for _ in range(n))
```



#### 4 turn_over 函数

该函数用于判定本轮回合是否结束，参数即投掷的骰子序列 `rolls`。如果玩家主动中止回合，则作为特殊情况直接 `break` 循环即可。于是有如下实现：

```python
def turn_over(rolls):
    '''
    Return True if the turn is over, False otherwise.
    
    The turn is over if any of the rolls is a 1, 
    or if there are exactly five rolls.
    '''
    return 1 in rolls or len(rolls) == 5
```



#### 5 take_turn 函数

有了 `roll_dice` 和 `turn_over` 两个子函数，就可以率先实现 `take_full_turn` 函数了。该函数接收奖池的初始筹码数，返回当前回合结束时的新奖池筹码、以及最终的骰子序列：

```python
def take_full_turn(pot_chips):
    '''
    The pot has pot_chips chips.
    
    Take a full turn for the current player and, once done, 
    return a list of two values:
    - the number of chips in the pot
    - the final list of dice rolls.
    
    Begin by rolling 1 die, and put 1 chip into the pot.
    Then, if the turn isn't over, ask the player whether 
        they'd like to continue their turn.
    If they respond 'n', then the turn is over.
    If they respond 'y', then roll one more die than last time, 
        and add 1 chip to the pot for each die that is rolled.
    (for example, if 3 dice were rolled last time, then 
        roll 4 dice and add 4 chips to the pot.)
    If the turn is not over, repeat by asking the player again 
        whether they'd like to continue their turn.
    '''
    rolls = roll_dice(1)
    pot_chips += 1
    while not turn_over(rolls):
        keep_going = input("Would you like to continue your turn? (y/n) ")
        if keep_going == 'n':
            break
        elif keep_going == 'y':
            rolls = roll_dice(len(rolls) + 1)
            pot_chips += len(rolls)
    return [pot_chips, rolls]
```

这样就又实现了三个关键节点：

![](assets/10.8.png)



#### 6 wins_chips 函数

如上图所示，该函数用于在当前回合结束后，根据投骰子结果判定玩家是出局退出的还是主动退出的，如果是后者，则需要将奖池筹码累加给该玩家，否则奖池筹码滚动到下一回合备用（即不作任何操作）：

```python
def wins_chips(rolls):
    '''
    Return True if the player wins chips, False otherwise.
    
    The player wins the chips if none of the rolls is a 1.
    '''
    return not 1 in rolls
```



#### 7 play 主函数的实现

至此，所有的子函数都已经实现完毕：

![](assets/10.9.png)

接下来需要根据游戏规则，将上述所有子函数嵌入游戏的主流程中：

```python
def play():
    '''
    Play the game until the game is over.
    
    The pot starts with 0 chips, and each player starts with 0 chips.
    
    Randomly decide whether player 1 or player 2 goes first.
    
    Before each turn, print three lines of information:
    1. The number of chips in the pot
    2. The number of chips that each player has
    3. Whether it is player 1's turn or player 2's turn
    
    Take a full turn for the current player. 
    If they won the chips, add the chips in the pot to the 
    total for that player and reset the pot to have 0 chips.
 
    Then, switch to the other player's turn.
 
    Once the game is over, print the current player 
    (that's the player who won).
    '''
    pot_chips = 0
    player1 = 0
    player2 = 0
    current_player = random.choice([1, 2])
    while not game_over(player1, player2, []):
        print(f"Pot has {pot_chips} chips.")
        print(f"Player 1 has {player1} chips.")
        print(f"Player 2 has {player2} chips.")
        print(f"It is Player {current_player}'s turn.")
        
        pot_chips, rolls = take_full_turn(pot_chips)
        
        if wins_chips(rolls):
            if current_player == 1:
                player1 += pot_chips
            else:
                player2 += pot_chips
            pot_chips = 0
        
        current_player = switch_player(player1, player2, rolls, current_player)
        
    print(f"Player {current_player} wins!")
```

最后在末尾调用 `play` 函数即可：

```python
if __name__ == "__main__":
    play()
```



### 10.4.5 实测 Bogart 小游戏

在命令行运行 `python demo.py` 启动该游戏，得到如下结果：

![](assets/10.10.png)

显然，游戏缺乏必要的中间结果输出。按照书中的提示自行补充完善，最终的实现版本如下：

```python
import random

def game_over(player1, player2, rolls):
    # -- snip --
    return player1 >= 30 or player2 >= 30 \
      or (len(rolls) == 5 and 1 not in rolls)
      
      
def switch_player(player1, player2, rolls, current_player):
    # -- snip --
    if not game_over(player1, player2, rolls):
        current_player = 1 if current_player == 2 else 2
    return current_player

def turn_over(rolls):
    # -- snip --
    return 1 in rolls or len(rolls) == 5
  
def roll_dice(n):
    # -- snip --
    rolls = list(random.randint(1, 6) for _ in range(n))
    print("Rolled:", rolls, ('' if wins_chips(rolls) else '(Aced out)'))
    return rolls
  
def take_full_turn(pot_chips):
    # -- snip --
    rolls = roll_dice(1)
    pot_chips += 1
    while not turn_over(rolls):
        keep_going = input("Would you like to continue your turn? (y/n) ")
        if keep_going == 'y':
            rolls = roll_dice(len(rolls) + 1)
            pot_chips += len(rolls)
            print(f"Pot now has {pot_chips} chips.")
        else:
            break
    return [pot_chips, rolls]
  
def wins_chips(rolls):
    # -- snip --
    return 1 not in rolls
  
def play():
    # -- snip --
    pot_chips = 0
    player1 = 0
    player2 = 0
    current_player = random.choice([1, 2])
    
    print('Welcome to Bogart!')
    print()
    
    turn = 0
    rolls = []
    while not game_over(player1, player2, rolls):
        print('--' * 10, f'Turn {turn + 1}', '--' * 10)
        print(f"Pot chips: {pot_chips}")
        print(f"Player 1 chips: {player1}")
        print(f"Player 2 chips: {player2}")
        print(f"Now it's Player {current_player}'s turn.")
        
        pot_chips, rolls = take_full_turn(pot_chips)
        
        if wins_chips(rolls):
            if(len(rolls) == 5):
                print('Rolled 5 dice without acing out! Automatically wins the pot!')
                break
            if current_player == 1:
                player1 += pot_chips
            else:
                player2 += pot_chips
            pot_chips = 0
        
        current_player = switch_player(player1, player2, rolls, current_player)
        turn += 1
    
    print()
    print()
    print('-=' * 20)     
    print()
    
    current_chips = player1 if current_player == 1 else player2
    print(f"Player {current_player} wins, with {current_chips} chips!")
    
if __name__ == "__main__":
    play()
```

实测结果：

```bash
python demo.py    
Welcome to Bogart!

-------------------- Turn 1 --------------------
Pot chips: 0
Player 1 chips: 0
Player 2 chips: 0
Now it's Player 2's turn.
Rolled: [2]
Would you like to continue your turn? (y/n) y
Rolled: [1, 5] (Aced out)
Pot now has 3 chips.
-------------------- Turn 2 --------------------
Pot chips: 3
Player 1 chips: 0
Player 2 chips: 0
Now it's Player 1's turn.
Rolled: [4]
Would you like to continue your turn? (y/n) y
Rolled: [3, 3] 
Pot now has 6 chips.
Would you like to continue your turn? (y/n) y
Rolled: [1, 1, 6] (Aced out)
Pot now has 9 chips.
-------------------- Turn 3 --------------------
Pot chips: 9
Player 1 chips: 0
Player 2 chips: 0
Now it's Player 2's turn.
Rolled: [6]
Would you like to continue your turn? (y/n) y
Rolled: [1, 3] (Aced out)
Pot now has 12 chips.
-------------------- Turn 4 --------------------
Pot chips: 12
Player 1 chips: 0
Player 2 chips: 0
Now it's Player 1's turn.
Rolled: [1] (Aced out)
-------------------- Turn 5 --------------------
Pot chips: 13
Player 1 chips: 0
Player 2 chips: 0
Now it's Player 2's turn.
Rolled: [4]
Would you like to continue your turn? (y/n) y
Rolled: [1, 6] (Aced out)
Pot now has 16 chips.
-------------------- Turn 6 --------------------
Pot chips: 16
Player 1 chips: 0
Player 2 chips: 0
Now it's Player 1's turn.
Rolled: [3]
Would you like to continue your turn? (y/n) n
-------------------- Turn 7 --------------------
Pot chips: 0
Player 1 chips: 17
Player 2 chips: 0
Now it's Player 2's turn.
Rolled: [3]
Would you like to continue your turn? (y/n) y
Rolled: [3, 4] 
Pot now has 3 chips.
Would you like to continue your turn? (y/n) y
Rolled: [3, 6, 5] 
Pot now has 6 chips.
Would you like to continue your turn? (y/n) y
Rolled: [2, 3, 3, 4] 
Pot now has 10 chips.
Would you like to continue your turn? (y/n) y
Rolled: [6, 4, 4, 1, 5] (Aced out)
Pot now has 15 chips.
-------------------- Turn 8 --------------------
Pot chips: 15
Player 1 chips: 17
Player 2 chips: 0
Now it's Player 1's turn.
Rolled: [4]
Would you like to continue your turn? (y/n) n


-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

Player 1 wins, with 33 chips!
```

> [!tip]
>
> **实测备忘**
>
> 原书代码存在逻辑漏洞：在完整回合结束后、累加奖池筹码前，还应该先确认第二个游戏结束条件：5 个骰子均不为 1 点。若满足，则直接结束游戏，跳过后续的累加逻辑。
>
> 因此第 `3.3` 步更正前后对比如下：
>
> ```python
> # before
> if wins_chips(rolls):
>     if current_player == 1:
>         player1 += pot_chips
>     else:
>         player2 += pot_chips
>     pot_chips = 0
> 
> # after
> if wins_chips(rolls):
>     if(len(rolls) == 5):
>         print('Rolled 5 dice without acing out! Automatically wins the pot!')
>         break
>     if current_player == 1:
>         player1 += pot_chips
>     else:
>         player2 += pot_chips
>     pot_chips = 0
> ```
>
> 这个 Bug 非常隐蔽，它出现在自顶向下的流程设计阶段（步骤 `3.3`）。作者本意是想强调问题拆分能力的重要性，结果偏偏在这里出了 Bug，直到后面实测才发现问题。那么，AI 能否在流程设计方面也做些工作呢？如果可以，AI 设计的流程应该如何验证有效性呢？现阶段可能还是只能依靠开发者手动审核验证，这样还不如不用 AI（或许可以从强化测试来解决这个问题，如 TDD、BDD 等）。





---

[^1]: 该参考资料是由作者 **A. Sweigart** 编写的《Invent Your Own Computer Games with Python》第 4 版，2016 年由 **No Starch Press** 出版发行。中译本《Python游戏编程快速上手 第4版》。

