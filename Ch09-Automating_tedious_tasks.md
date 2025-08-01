# Ch09: Automating tedious tasks

> **本章摘要**
>
> - 程序员编写工具的原因
> - 确定工具所需的模块
> - 示例1：自动清理电子邮件中的缩进符号 `>`
> - 示例2：自动操作 `PDF` 文件
> - 示例3：自动删除多个图片库中的重复图片

本章将从三个典型的日常低效问题入手，进一步演示 `ChatGPT` 这样的 `AI` 工具在工具脚本编写方面的落地应用，并结合具体的演示情况进行评价。从这一章开始，介绍的内容实操性都很强，建议模仿流程进行实践演练。



## 1 编写工具的原因

程序员经常自嘲说自己很 **懒**，背后的潜台词其实只是 **不想重复造轮子**（即践行 `DRY` 原则，`Don't Repeat Yourself`）罢了。每当遇到一些高频低效的机械型重复劳动，其敏锐的职业第六感（`spidey-sense`）会将该任务快速抽象为一个相对通用的脚本处理流程，并通过某个脚本语言进行实现，从而形成某个脚本工具，达到解放双手，**一键到位** 的目的。

这里其实有个不大不小的坑：低效重复究竟要达到什么程度才值得动手编写工具脚本呢？

很遗憾，这个问题并没有放之四海而皆准的标准答案，传统路径通常少不了要苦哈哈地手动尝试几轮，直到某一天实在受不了这样的低效重复了，才开始慢慢总结规律，最后诉诸脚本；而如今有了 `AI` 工具，前期的探索阶段就可以大幅缩减，最终只需要预判一下这个任务今后是否会经常执行就可以了，从 0 到 1 的脚本实现过程自此也就相对不那么望而生畏了。



## 2 确定工具所需的模块

面对陌生的任务场景，确定脚本工具的依赖项成了新的拦路虎。以前这类问题都是通过搜索引擎在各大技术论坛中大海捞针，获取灵感后再改造成自己需要的代码逻辑。现在这个过程也简化了，在 `VSCode` 的 `Copilot chat` 模块利用直接对话就能知道想要的内容，连知识迁移这部都省了，取而代之的是开发者的提问方式和问题整合能力。这样的过渡究竟算不算进步现在下结论还为时尚早，因为大脑从某种程度上可能并不认可这样的“减负”，对某个领域的认知深度从此或将完全取决于提问的质量和大模型的训练结果，颇有点被大模型驯化的味道，让人细思极恐。

扯远了，回到正题。本章主要还是从构建基于 `Python` 脚本的工具函数进行演示，因为函数具备天然的灵活性，可以很好地解决后续扩展的问题（扩充参数列表就行了）。这里选取了三个不同的典型案例，接下来逐一进行考察。



## 3 示例1：自动清理电子邮件中的缩进符号

### 3.1 明确需求

如果电子邮件回复次数较多，新邮件就会自动产生一些缩进符号来标识每一轮交谈的内容：

```markdown
> > > Hi Leo,
> > > > > Dan -- any luck with your natural language research?
> > > Yes! That website you showed me
https://www.kaggle.com/
> > > is very useful. I found a dataset on there that collects
a lot
> > > of questions and answers that might be useful to my research.
> > > Thank you,
> > > Dan
```

其实在国内类似的场景并不多见，工作交流有一大堆现成的工具：企微、QQ、钉钉……根本轮不到电子邮件登场，更何况这么标准的格式可能还得手动设置为纯文本模式才能看到；但作为辅助编程在日常办公中的第一个案例，关注演示的重点即可。最终想要的处理结果如下：

```markdown
Hi Leo,
Dan -- any luck with your natural language research?
Yes! That website you showed me
https://www.kaggle.com/
is very useful. I found a dataset on there that collects
a lot
of questions and answers that might be useful to my research.
Thank you,
Dan
```

按作者的思路，原始文本已经复制到了剪贴板（例如通过 `Ctrl+C`），然后执行该工具脚本，再粘贴出来就是处理后的结果了（有点反人类，忍忍吧）。

### 3.2 方案海选

为此，需要与 `Github Copilot` 多轮交互，从多个候选方案中择优。先给出我们的需求：

```markdown
I would like to write a Python program that accesses the clipboard and lets me copy and paste using the clipboard. How can I do this?
```

当前的 `GPT-4.1` 模型（2025-8-1）给我的回复是：

![](assets/9.1.png)

**图 9-1：Github Copilot 最新模型的实测回复内容**

虽然这个方案解决了跨平台的兼容性问题，但由于我们不想引入第三方模块，于是继续提问：

```markdown
Is there a way I can do it without having to install a Python module?
```

此时 `Copilot` 根据我当前的 Windows 系统，直接跳过了书中演示的两个中间环节，给出了基于 `subprocess` 标准模块的第二版（说明大模型一直在更新，且越来越智能了）：

> Yes, you can use built-in modules and system commands, but it depends on your operating system.
>
> **On Windows**, you can use `subprocess` with `clip` (copy) and `powershell Get-Clipboard` (paste):
>
> ```python
> import subprocess
> 
> # Copy to clipboard
> def copy_to_clipboard(text):
>     process = subprocess.Popen(['clip'], stdin=subprocess.PIPE, close_fds=True)
>     process.communicate(input=text.encode('utf-8'))
> 
> # Paste from clipboard
> def paste_from_clipboard():
>     result = subprocess.run(
>         ['powershell', '-command', 'Get-Clipboard'],
>         capture_output=True, text=True
>     )
>     return result.stdout.strip()
> 
> # Example usage
> copy_to_clipboard("Hello, clipboard!")
> print("Clipboard contains:", paste_from_clipboard())
> ```
>
> No extra modules needed—just Python’s standard library.

但是，我们不想让脚本与平台绑定，因此再次追问：

```markdown
Does that work on Linux or macOS too?
```

这一回 `Copilot` 又分别给出了提到的两个平台的具体脚本实现，比原书中的概括性描述精准太多了：

> No, the previous example is specific to Windows.
>
> **For Linux**, you can use `xclip` or `xsel` (these must be installed):
>
> ```python
> import subprocess
> 
> def copy_to_clipboard(text):
>     subprocess.run(['xclip', '-selection', 'clipboard'], input=text.encode('utf-8'))
> 
> def paste_from_clipboard():
>     result = subprocess.run(['xclip', '-selection', 'clipboard', '-o'], capture_output=True)
>     return result.stdout.decode('utf-8')
> 
> # Example usage
> copy_to_clipboard("Hello, clipboard!")
> print("Clipboard contains:", paste_from_clipboard())
> ```
>
> **For macOS**, use `pbcopy` and `pbpaste`:
>
> ```python
> import subprocess
> 
> def copy_to_clipboard(text):
>     subprocess.run(['pbcopy'], input=text.encode('utf-8'))
> 
> def paste_from_clipboard():
>     result = subprocess.run(['pbpaste'], capture_output=True)
>     return result.stdout.decode('utf-8')
> 
> # Example usage
> copy_to_clipboard("Hello, clipboard!")
> print("Clipboard contains:", paste_from_clipboard())
> ```
>
> These methods require the respective clipboard utilities to be available on your system.

经过上述三轮探索，权衡下来安装 `pyperclip` 第三方模块的版本倒也不是那么一无是处，于是就定它了。

需要注意的是，**方案海选** 这一步看似简单，实际操作中变数最大，耗费的时间精力可能远比传统方式多得多：因为大模型的本质是一个概率模型，受模型的升级、模块的更新换代等因素的影响，同一个问题可能得到完全不同的回复内容；评价这些方案时，要是再缺少相应的技术储备，在最糟糕的情况下可能真得挨个试一遍，想想头皮都发麻……



### 3.3 脚本实现

接下来是具体的脚本实现。首先安装第三方模块：

```python
pip install pyperclip
```

然后按照前面介绍的固定结构，通过声明函数签名和 `docstring` 文档字符串，引导 `Copilot` 按 `pyperclip` 的版本实现具体代码。最终结果如下（从 `L10` 开始）：

```python
import pyperclip
 
def clean_email():
    '''
    The clipboard contains lines of text.
    Clean up the text by removing any > or space
    characters from the beginning of each line.
    Replace the clipboard with the cleaned text.
    '''
    text = pyperclip.paste()
    lines = text.split('\n')
    cleaned_lines = []
    for line in lines:
        cleaned_line = line.lstrip('> ').strip()
        cleaned_lines.append(cleaned_line)
    cleaned_text = '\n'.join(cleaned_lines)
    pyperclip.copy(cleaned_text)
    print("Cleaned text copied to clipboard.")
    
if __name__ == "__main__":
    clean_email()
    
# Example usage:
# 1. Copy some text to the clipboard that contains lines starting with '>' or spaces    
# 2. Run this script
# 3. The cleaned text will be copied back to the clipboard
```

复制最开始的原始文本，然后用 `python demo.py` 执行该脚本：

```cmd
> python demo.py
```

再用 <kbd>Ctrl</kbd> + <kbd>V</kbd> 粘贴到某个空白文本框进行验证，居然真的直接就搞定了：

![](assets/9.2.png)

**图 9-2：最终工具脚本的测试验证结果（通过测试）**

得益于大模型的升级，原书中的报错与代码调试环节也省了，但这并不能说明今后的回复内容都这么完美，切莫掉以轻心。真遇到问题了也只能具体分析。

这个案例只能算开胃菜，还有很多细节有待完善，这里就不展开了。最后再总结一下几个要点：

1. 明确需求：准确描述想要的最终效果；
2. 方案海选：多问几次 `Copilot`，并根据提供的多个方案进行综合评估，最终敲定技术路线；
3. 工具实现：按照选好的方案实现脚本，引导 `Copilot` 在具体的函数签名下，基于 `docstring` 的描述生成最终脚本；
4. 脚本测试与调试：测试如果遇阻，可继续追问 `Copilot`，直到顺利实现既定效果；
5. 脚本优化：根据实际情况酌情完善细节，但切忌过于追求完美。
