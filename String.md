# 字符串

## 串联

方法一（不推荐）：

```
$message = "Hello, $first $last."
```
在下面这种情况下，将无法达到预期

```
PS> $directory = Get-Item 'c:\windows'
PS> $message = "Time: $directory.CreationTime"
PS> $message

Time: C:\windows.CreationTime
```
方法二（不推荐，但可以实现方法一种的问题）

```
$message = "Date: $(Get-Date)"
```
或者

```
$message = "Time: $($directory.CreationTime)"
```

方法三(推荐)

```
"{0:yyyyMMdd}" -f (Get-Date)
"Population {0:N0}" -f  8175133
```

## StringBuilder

对于使用许多较小的字符串生成大型字符串，StringBuilder 也非常受欢迎。 原因是，它只收集你向它添加的所有字符串，并且只在你检索值时在末尾串联所有字符串

```
$stringBuilder = New-Object -TypeName "System.Text.StringBuilder"

[void]$stringBuilder.Append("Numbers: ")
foreach($number in 1..10000)
{
    [void]$stringBuilder.Append(" $number")
}
$message = $stringBuilder.ToString()
```

## 查找和替换标记

```
$letter = Get-Content -Path TemplateLetter.txt -RAW
$letter = $letter -replace '#FULL_NAME#', 'Kevin Marquette'
```

## 替换多个标记

```
$tokenList = @{
    Full_Name = 'Kevin Marquette'
    Location = 'Orange County'
    State = 'CA'
}

$letter = Get-Content -Path TemplateLetter.txt -RAW
foreach( $token in $tokenList.GetEnumerator() )
{
    $pattern = '#{0}#' -f $token.key
    $letter = $letter -replace $pattern, $token.Value
}
```

## ExecutionContext ExpandString

有一种巧妙的方法可以使用单引号定义替换字符串，之后再展开变量。 请看下面的示例：

```
$message = 'Hello, $Name!'
$name = 'Kevin Marquette'
$string = $ExecutionContext.InvokeCommand.ExpandString($message)
```
对当前执行上下文的 .InvokeCommand.ExpandString 调用使用了当前作用域内的变量进行替换。 此处的关键在于，可以在变量存在之前尽早定义 $message。

如果我们再展开一点点，就可以使用不同的值反复执行此替换。

```
$message = 'Hello, $Name!'
$nameList = 'Mark Kraus','Kevin Marquette','Lee Dailey'
foreach($name in $nameList){
    $ExecutionContext.InvokeCommand.ExpandString($message)
}
```