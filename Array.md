# Array数组

```
$data=@()
$data01=@(1,2,3,4)
$data02=@('Cp1','Cp2','Cp3')
$data03=1,2,3

PS> $data = 'Zero','One','Two','Three'
PS> $data[0]
Zero

PS> $data[0,2,3]
Zero
Two
Three

PS> $data[1..3]
One
Two
Three

PS> $data[3..1]
Three
Two
One

PS> $data[-1]
Three

PS> $a = 1,2,3,4,5,6,7,8
PS> $a[2..-1]
3
2
1
8

PS> $data[ $data.count - 1 ]
Three

PS> $data.GetUpperBound(0)
3
PS> $data[ $data.GetUpperBound(0) ]
Three
```

**更新**

```
$data[2] = 'dos'
$data[3] = 'tres'
```

**迭代**

数组和 PowerShell 管道是相互适用的。 这是处理这些值最简单的方法之一。 将数组传递给管道时，数组中的每个项将得到单独处理

```
PS> $array=1,2,3,4,5,6
PS> $array |% {$_}
1
2
3
4
5
6

PS> $array |%{ "item:[$PsItem]"}
item:[1]
item:[2]
item:[3]
item:[4]
item:[5]
item:[6]
```

***ForEach foreach For***

PowerShell 允许你对集合调用 .ForEach()。

```
PS> $data.foreach({"Item [$PSItem]"})
Item [Zero]
Item [One]
Item [Two]
Item [Three]

$data.foreach{"Item [$PSItem]"}

for ( $index = 0; $index -lt $data.count; $index++)
{
    "Item: [{0}]" -f $data[$index]
}

PS> for($i=0;$i -lt $array.Count;$i++){
>> $array[$i]
>> }
1
2
3
4
5
6

for ( $index = 0; $index -lt $data.count; $index++ )
{
    $data[$index] = "Item: [{0}]" -f $data[$index]
}
```

***对象数组***

```
PS> $data = @(
>>     [pscustomobject]@{FirstName='Kevin';LastName='Marquette'}
>>     [pscustomobject]@{FirstName='John'; LastName='Doe'}
>> )

PS D:\Desktop\PoweshellOfScatteredBeans> $data.Count
2

PS D:\Desktop\PoweshellOfScatteredBeans> $data[0].LastName
Marquette

PS> $data | ForEach-Object {$_.LastName}
Marquette
Doe

PS> $data | Where-Object {$_.FirstName -eq 'Kevin'}
FirstName LastName
-----     ----
Kevin     Marquette


$data | Where FirstName -eq Kevin

```

***运算符***

**-join**

```
PS> $data = @(1,2,3,4)
PS> $data -join '-'
1-2-3-4
PS> $data -join ','
1,2,3,4

```

**-join $array**

***<font size=7>[妙招]</font>*** 如果你希望无需分隔符即可联接所有内容，请不要这样做：

```
PS> $data = @(1,2,3,4)
PS> $data -join $null
1234
```
可以将数组作为不带前缀的参数与 -join 一起使用。 具体请看下面的示例。

```
PS> $data = @(1,2,3,4)
PS> -join $data
1234
```

**-replace 和 -split**

```
PS> $data = @('ATX-SQL-01','ATX-SQL-02','ATX-SQL-03')
PS> $data -replace 'ATX','LAX'
LAX-SQL-01
LAX-SQL-02
LAX-SQL-03
```

**-contains**
```
PS> $data = @('red','green','blue')
PS> $data -contains 'green'
True
```

***<font size=6>-in</font>***
```
PS> $data = @('red','green','blue')
PS> 'green' -in $data
True
```

***<font size=6>-eq 和 -ne</font>***

```
PS> $data = @('red','green','blue')
PS> $data -eq 'green'
green

PS> $data = @('red','green','blue')
PS> $data -ne 'green'
red
blue
```

在 if() 语句中使用此运算符时，返回的值为 True 值。 如果未返回任何值，则它是 False 值。 下面这两个语句的计算结果都是 True

```
$data = @('red','green','blue')
if ( $data -eq 'green' )
{
    'Green was found'
}
if ( $data -ne 'green' )
{
    'And green was not found'
}
```

***<font size=6>$null 或空</font>***

测试 $null 或空数组可能比较棘手。 下面是一些常见的数组陷阱。

这个语句乍一看似乎可行。

```
if ( $array -eq $null)
{
    'Array is $null'
}
```
但我刚刚讲了 -eq 如何检查数组中的每一项。 因此，我们可以有一个由几个项组成的数组，其中只有一个 $null 值，它的值将为 $true

```
$array = @('one',$null,'three')
if ( $array -eq $null)
{
    'I think Array is $null, but I would be wrong'
}
```

这就是最好将 $null 放置在运算符左侧的原因。 这样此方案将不会出现任何问题。
```
if ( $null -eq $array )
{
    'Array actually is $null'
}
```
$null 数组与空数组不同。 如果知道有一个数组，请检查其中对象的计数。 如果数组为 $null，则计数为 0。
```
if ( $array.count -gt 0 )
{
    "Array isn't empty"
}
```
还有一个陷阱需要注意。 即使有单个对象，也可以使用 count，除非该对象是 PSCustomObject。 这是在 PowerShell 6.1 中修复的 bug。 这是个不错的消息，但有很多人仍在使用 5.1，因此需要注意。
```
PS> $object = [PSCustomObject]@{Name='TestObject'}
PS> $object.count
$null
```
如果仍在使用 PowerShell 5.1，则可以在检查计数之前将对象包装在数组中，以获取准确的计数。
```
if ( @($array).count -gt 0 )
{
    "Array isn't empty"
}
```
稳妥起见，请检查是否有 $null，然后再检查计数。
```
if ( $null -ne $array -and @($array).count -gt 0 )
{
    "Array isn't empty"
}
```

***待续。。。***