# PSCustomObject 

创建 

```
$myObject = [PSCustomObject]@{
    Name     = 'Kevin'
    Language = 'PowerShell'
    State    = 'Texas'
}
```

转换哈希表

```
$myHashtable = @{
    Name     = 'Kevin'
    Language = 'PowerShell'
    State    = 'Texas'
}

$myObject = [pscustomobject]$myHashtable
```

保存到文件

```
$myObject | ConvertTo-Json -depth 1 | Set-Content -Path $Path
$myObject = Get-Content -Path $Path | ConvertFrom-Json
```

添加属性

```
$myObject | Add-Member -MemberType NoteProperty -Name 'ID' -Value 'KevinMarquette'

$myObject.ID
```
删除属性

```
$myObject.psobject.properties.remove('ID')
```

枚举属性名称
有时需要对象上所有属性名称的列表。

```
$myObject | Get-Member -MemberType NoteProperty | Select -ExpandProperty Name
```

将 PSCustomObject 转换为哈希表
要从上一节继续操作，可以动态地遍历属性并从中创建一个哈希表。

```
$hashtable = @{}
foreach( $property in $myobject.psobject.properties.name )
{
    $hashtable[$property] = $myObject.$property
}
```