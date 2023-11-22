# panda-dto

php数据传输对象

## 安装

```bash
composer require pandaxxw/panda-dto
```

* **注意**: 要求至少>=PHP 7.4

## 如何使用

```php
use App\Models\Author;
use Spatie\DataTransferObject\DataTransferObject;

class PostData extends Dto
{
    public string $title;
    
    public string $body;
    
    public Author $author;
}
```

初始化对象

```php
$postData = new PostData([
    'title' => '…',
    'body' => '…',
    'author_id' => '…',
]);
```

现在就可以按对象使用了

```php
$postData->title;
$postData->body;
$postData->author_id;
```

```php
use App\Models\Author;
use Iterator;
use Pandaxxw\Dto\Dto;

class PostData extends Dto
{
    /**
     * Built in types:
     */
    public string $property;

    /**
     * Imported class or fully qualified class name:
     */
    public Author $property;

    /**
     * Nullable types:
     */
    public ?string $property;
    
    /**
     * Any iterator:
     */
    public Iterator $property;
    
    /**
     * No type, which allows everything
     */
    public $property;
}
```

### 集合

```php

class PostCollection extends \Pandaxxw\Dto\DtoCollection
{
    public function current(): PostData
    {
        return parent::current();
    }
}
```

通过重写该current方法，将自动完成功能：
然后你可以像这样使用集合：

```php
foreach ($postCollection as $postData) {
    $postData->
}

$postCollection[0]->
```

您可以自由地实现自己的静态构造函数：

```php
use App\DataTransferObjects\PostData;
use Spatie\DataTransferObject\DataTransferObjectCollection;

class PostCollection extends \Pandaxxw\Dto\DtoCollection
{
    public static function create(array $data): PostCollection
    {
        return new static(PostData::arrayOf($data));
    }
}
```

### 自动嵌套 DTO

如果您有嵌套的 DTO 字段，则传递到父 DTO 的数据将自动进行转换。

```php

class PostData extends Dto
{
    public AuthorData $author;
}
```

`PostData` 现在可以像这样构建:

```php
$postData = new PostData([
    'author' => [
        'name' => 'Foo',
    ],
]);
```

**Attention**: Remember, for nested type casting to work, your Docblock definition needs to be a Fully Qualified Class
Name (`\App\DataTransferObjects\TagData[]` instead of `TagData[]` and a use statement at the top).

### 不可改变的Dto

如果你要某个Dto初始化之后不能更改，可以用它来初始化

```php
$postData = PostData::immutable([
    'tags' => [
        ['name' => 'foo'],
        ['name' => 'bar']
    ]
]);
```

如果改变$postData的属性, 将抛 `DtoError`.

### 辅助函数

```php
$postData->all();

$postData
    ->only('title', 'body')
    ->toArray();
    
$postData
    ->except('author')
    ->toArray();
``` 

可以用链式调用:

```php
$postData
    ->except('title')
    ->except('body')
    ->toArray();
```

需要注意的是，except和only是不可变的，它们不会改变原始数据传输对象。

### 异常处理

除了属性类型验证之外，您还可以确定整个数据传输对象始终有效。在构造数据传输对象时，我们将验证是否设置了所有必需的（不可为空）属性。如果没有，Spatie\DataTransferObject\DataTransferObjectError将会抛出
a 。

同样，如果您尝试设置未定义的属性，Dto.

### 灵活的数据传输对象

有时您可能希望能够使用数组的子集实例化 DTO。一个很好的例子是大型 API 响应，其中仅使用少量字段。通常，如果您尝试实例化具有多余属性的标准
DTO，DtoError则会抛出异常。

您可以通过FlexibleDto避免异常。例如：

```php

class PostData extends \Pandaxxw\Dto\FlexibleDto
{
    public string $content;
}


// No errors thrown
$dto = new PostData([
    'author' => [
        'id' => 1,
    ],
    'content' => 'blah blah',
    'created_at' => '2020-01-02',
]);

$dto->toArray(); // ['content' => 'blah blah']
```