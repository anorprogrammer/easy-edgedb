# Chapter 6 Questions and Answers

#### 1. 下面这个选择是不完整的。如何修改它从而使它能打印出“Pleased to meet you, I'm ”以及 NPC 的名字？

使用操作符 `++` 进行级联:

```edgeql
SELECT NPC {
  name,
  greeting := "Pleased to meet you, I'm " ++ .name
};
```

#### 2. 如果米娜要去德古拉城堡参观，你会如何更新米娜（Mina）的 `places_visited`，让它也包括罗马尼亚？

下面是一种方法：

```edgeql
UPDATE Person FILTER .name = 'Mina Murray'
SET {
  places_visited += (SELECT Place FILTER .name = 'Romania')
};
```

如果你喜欢，你也可以用 `UPDATE NPC` 和 `SELECT Country`。

此外，你也可以使用 `WITH` 达到同样的效果：

```edgeql
WITH
  mina := (SELECT NPC FILTER .name = 'Mina Murray'),
  romania := (SELECT Country FILTER .name = 'Romania'),
UPDATE mina
SET {
  places_visited += romania
};
```

#### 3. 如何显示出名称（name）里包含 `{'W', 'J', 'C'}` 中任何一个大写字母的所有 `Person` 对象？

答案是：

```edgeql
WITH letters := {'W', 'J', 'C'}
SELECT Person {
  name
} FILTER .name LIKE '%' ++ letters ++ '%';
```

会显示截止到现在我们已经插入的以下角色：

```
{
  Object {name: 'Woman 1'},
  Object {name: 'Woman 2'},
  Object {name: 'Woman 3'},
  Object {name: 'Jonathan Harker'},
  Object {name: 'Count Dracula'},
}
```

关键点是 `LIKE` 需要一个字符串，所以你可以用 `++` 将左右的 `%` 连接起来。

#### 4. 如何用 JSON 显示和上一题相同的查询？

用 `<json>` 进行转换可以很容易得到 JSON 输出，但是应该放在哪里呢？你不能放在 `SELECT` 的前面，也不能用 `<json>Person`，因为它不是一个表达式，所以像下面这样是无法工作的：

```edgeql
WITH letters := {'W', 'J', 'C'}
SELECT <json>Person {
  name
} FILTER .name LIKE '%' ++ letters ++ '%';
```

你需要把 SELECT 包装到小括号里，然后用 `<json>` 进行转换，并 SELECT 它：

```edgeql
WITH letters := {'W', 'J', 'C'}
SELECT <json>(
  SELECT Person {
    name
  } FILTER .name LIKE '%' ++ letters ++ '%'
);
```

你也可以像下面这样做：

```edgeql
WITH
  letters := {'W', 'J', 'C'},
  P := (
    SELECT Person FILTER .name LIKE '%' ++ letters ++ '%'
  )
SELECT <json>P { name };
```

如此，你正在选择 `SELECT Person` 被强制转换为 JSON 版本的结果。

#### 5. 如何将“ the Greate”添加到每个 Person 类型的对象中？

很简单，只需在没有 `FILTER` 的情况下对类型进行更新：

```edgeql
UPDATE Person
SET {
  name := .name ++ ' the Great'
};
```

现在她们的名字是：“Woman 1 the Great”, “Mina Murray the Great”等等。

**此外**: 使用字符串索引来撤销上述操作的快速方法是用 `[0:-10]` 去掉 `name` 字符串后十位字符并再赋值给 `name`。

```edgeql
UPDATE Person
SET {
  name := .name[0:-10]
};
```
