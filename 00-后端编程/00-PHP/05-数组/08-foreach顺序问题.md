PHP的数组保存元素的顺序是根据元素插入的顺序来保存的。

```php
$arr = array(
    1 => 'a',
    0 => 'b',
    2 => 'c'
);
foreach ($arr as $val)
{
    echo $val . ' '; // a b c
}

```



