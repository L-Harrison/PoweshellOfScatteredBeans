## 哈希表

哈希区别于数组，其定义为 $hash=@{}  采用大括号。数组采用括号() eg: $array=@()

哈希表是一种数据结构，与数组非常相似，只不过是使用键存储每个值（对象）。 它是一个基本的键/值存储。

```
$key = 'Kevin'
$value = 36
$ageList.add( $key, $value )

$ageList.add( 'Alex', 9 )

$ageList = @{
    Kevin = 36
    Alex  = 9
}
```

循环访问哈希表

```
PS> $ageList | Measure-Object
count : 1


PS>$ht
Name                           Value
----                           -----
王松                           浪潮经理
张涛                           浪潮销售主管


PS> $ht.Keys|%{ '员工{0}所在的职务是{1}' -f $_,$ht[$_]}
员工王松所在的职务是浪潮经理
员工张涛所在的职务是浪潮销售主管
PS> $ht.GetEnumerator()|%{
>> '员工{0}所在的职务是{1}' -f $_.Key,$_.Value}
员工王松所在的职务是浪潮经理
员工张涛所在的职务是浪潮销售主管
```

***<font size=5>一个重要的细节是，哈希表在枚举时无法修改。</font>*** 可用下面技巧实现：

```
PS> $ht.Clone().GetEnumerator()|
>> ?{$_.Key -eq '王松'}|
>> %{$ht[$_.Key]='销售副总'}
PS> $ht

Name                           Value
----                           -----
张涛                           浪潮销售主管
王松                           销售副总
```

**哈希表作为属性集合**

可以像对象访问属性那样访问哈希，如下：

```
$person = @{
    name = 'Kevin'
    age  = 36
}

$person.city = 'Austin'
$person.state = 'TX'
```

删除和清除键

```
$person.remove('age')
```

向它们赋值 $null 后只会留下一个具有 $null 值的键。

清除哈希表的常用方法是将其初始化为空哈希表。

```
$person = @{}
```

虽然这样做确实有用，但请尝试改用 clear() 函数。

```
$person.clear()
```

***排序哈希表***

默认情况下，哈希表不进行排序（或排列）。 在传统的上下文中，当你始终使用键来访问值时，顺序并不重要。 你可能会发现，你希望属性保持你为它们定义的顺序。 令人欣慰的是，有一种方法可以使用 ordered 关键字实现此目的。当你枚举键和值时，它们会保持该顺序。

```
$person = [ordered]@{
    name = 'Kevin'
    age  = 36
}
```

***常见管道命令中的自定义表达式***

```
$property = @{
    name = 'totalSpaceGB'
    expression = { ($_.used + $_.free) / 1GB }
}

$drives = Get-PSDrive | Where Used
$drives | Select-Object -Property name, $property

Name     totalSpaceGB
----     ------------
C    238.472652435303

```

我将它放在一个变量中，但它可以很容易地在内联中定义，并且你可以顺便将 name 缩写为 n，将 expression 缩写为 e。

```
$drives | Select-Object -property name, @{n='totalSpaceGB';e={($_.used + $_.free) / 1GB}}
```

自定义排序表达式

如果对象具有你想要排序的数据，则可以很容易地对集合进行排序。 可以在排序之前将数据添加到对象，或者为 Sort-Object 创建自定义表达式。

```
Get-ADUser | Sort-Object -Property @{ e={ Get-TotalSales $_.Name } }
```

对一列哈希表排序

如果你有一列想要排序的哈希表，你会发现 Sort-Object 不会将键视为属性。 我们可以通过使用自定义排序表达式来解决。

```
$data = @(
    @{name='a'}
    @{name='c'}
    @{name='e'}
    @{name='f'}
    @{name='d'}
    @{name='b'}
)

$data | Sort-Object -Property @{e={$_.name}}
```

在执行 cmdlet 时展开哈希表

这是我最喜欢哈希表的地方之一，很多人以前都没有发现这一点。 其思路是，无需在一行上向 cmdlet 提供所有属性，而是先将它们打包为一个哈希表。 然后，你可以通过一种特殊方式将哈希表赋予函数。 下面是以常规方式创建 DHCP 作用域的示例。

```
Add-DhcpServerV4Scope -Name 'TestNetwork' -StartRange '10.0.0.2' -EndRange '10.0.0.254' -SubnetMask '255.255.255.0' -Description 'Network for testlab A' -LeaseDuration (New-TimeSpan -Days 8) -Type "Both"
```
如果不使用展开，则需要在一行上定义所有这些内容。 它会在屏幕上滚动或自动换行。 现在把它与使用展开的命令进行比较。

```
$DHCPScope = @{
    Name          = 'TestNetwork'
    StartRange    = '10.0.0.2'
    EndRange      = '10.0.0.254'
    SubnetMask    = '255.255.255.0'
    Description   = 'Network for testlab A'
    LeaseDuration = (New-TimeSpan -Days 8)
    Type          = "Both"
}
Add-DhcpServerV4Scope @DHCPScope
```
<font size=6>使用 @ 符号而非 $ 会调用展开操作。</font>

```
$Common = @{
    SubnetMask  = '255.255.255.0'
    LeaseDuration = (New-TimeSpan -Days 8)
    Type = "Both"
}

$DHCPScope = @{
    Name        = 'TestNetwork'
    StartRange  = '10.0.0.2'
    EndRange    = '10.0.0.254'
    Description = 'Network for testlab A'
}

Add-DhcpServerv4Scope @DHCPScope @Common
```

***添加哈希表***

哈希表支持通过加法运算符来合并两个哈希表。

```
$person += @{Zip = '78701'}
```

***嵌套哈希表***

```
$person = @{
    name = 'Kevin'
    age  = 36
}
$person.location = @{}
$person.location.city = 'Austin'
$person.location.state = 'TX'

$person = @{
    name = 'Kevin'
    age  = 36
    location = @{
        city  = 'Austin'
        state = 'TX'
    }
}
```

***<font size=6>创建对象</font>***

有时，只需有一个对象，而使用哈希表保存属性无法达到目的。 最常见的情况是，你想要将键视为列名。 pscustomobject 会让这变得简单。

```
$person = [pscustomobject]@{
    name = 'Kevin'
    age  = 36
}

$person

name  age
----  ---
Kevin  36
```

即使最初并未将其创建为 pscustomobject，也可以在以后需要时对其进行强制转换。

```
$person = @{
    name = 'Kevin'
    age  = 36
}

[pscustomobject]$person

name  age
----  ---
Kevin  36
```

**在文件中读取和写入哈希表**

将哈希表保存到 CSV 是我上面提到的难题之一。 将哈希表转换为 pscustomobject，它将正确保存到 CSV。 如果你从 pscustomobject 开始，这会很有帮助，这样可以保留列顺序。 但如果需要，可以将其强制转换为 pscustomobject 内联。

```
$person | ForEach-Object{ [pscustomobject]$_ } | Export-CSV -Path $path
```

**将嵌套哈希表保存到文件**

如果需要将嵌套哈希表保存到文件，然后再次将其读入，我会使用 JSON cmdlet。

```
$people | ConvertTo-JSON | Set-Content -Path $path
$people = Get-Content -Path $path -Raw | ConvertFrom-JSON
```

此方法有两个重要方面。 首先，JSON 采用多行编写，因此我需要使用 -Raw 选项将其读回到单个字符串中。 其次，导入的对象不再是 [hashtable]。 它现在是 [pscustomobject]，如果你不希望是这样，可能会导致问题。

如果需要在导入时为 [hashtable]，则需要使用 Export-CliXml 和 Import-CliXml 命令。

***待续。。。***