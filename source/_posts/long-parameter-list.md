---
title: 过长参数列（Long Parameter List）
date: 2021-03-22 00:13:30
tags:
---

# 过长参数列（Long Parameter List）

## 症状与体征
一个方法有三个或四个以上的参数。

<!-- more -->

## 问题原因

在一个方法中运用了多种不一样的算法后有可能会导致过长参数列。也就说，过长参数列可能为了控制运行哪种算法以及如何运行而被创建。
过长参数列也有可能是为了使类之间更加独立而产生的副作用。例如，用于创建方法中所需的特定对象的代码已从该方法移至用于调用该方法的代码，但是创建的对象作为参数传递给该方法。因此，原始类不再了解对象之间的关系，并且依赖性降低了。但是，如果创建了这些对象中的几个，则每个对象都将需要自己的参数，这意味着需要更长的参数列表。

## 疗法

* Replace Parameter with Method Call(以函数取代参数)
* Preserve Whole Object（保持对象完整）
* Introduce Parameter Object（引入参数对象）

### Replace Parameter with Method Call(以函数取代参数)

#### 解决思路
将参数在目标方法内部获取，以便减少当前方法参数数量

不好的示例
```php
$basePrice = $this->quantity * $this->itemPrice;
$seasonDiscount = $this->getSeasonalDiscount();
$fees = $this->getFees();
$finalPrice = $this->discountedPrice($basePrice, $seasonDiscount, $fees);
```

好的示例
```php
$basePrice = $this->quantity * $this->itemPrice;
$finalPrice = $this->discountedPrice($basePrice);
```



#### 重构步骤

1. 如果有必要，将参数的计算过程提炼到独立函数中
2. 将函数本体内引用该参数的地方改为调用新建的函数
3. 每次替换后，修改并测试
4. 全部替换完成后，使用`Remove Parameter` 将该参数去掉


#### 示例

```javascript
// edusoho/on-line-course/pages/task/taskStudent.js 202~210

    const menuData = functionButtomMenuData({
      role: data.taskData.role,
      courseId,
      lessonId,
      ingLessonId: data.study.lessonId,
      lessonStatus: data.study.lessonStatus,
      up: data.taskData.up,
      next: data.taskData.next,
    });
```


### Replace Parameter with Method Call(以明确函数取代参数)

```php
// src/Biz/Activity/Service/Impl/ActivityServiceImpl.php 243

protected function syncActivityMaterials($activity, $materials, $mode = 'create')

```

#### 重构步骤

1. 针对每个可能值，新建明确地函数
2. 修改每个调用分支，使其合适新的函数
3. 测试修复
4. 完毕后删除原函数

### Introduce Parameter Object（引入参数对象）

```javascript

// edusoho/pages/binding/binding.js 158

      userController('loginUser', password,identify,openId ).success(data=> {
      
      


```
[PHP示例](http://coding.codeages.work/edusoho/edusoho/-/blob/release/8.1.10/src/Biz/User/Service/Impl/UserServiceImpl.php)

https://wiki.codeages.work/pages/viewpage.action?pageId=17204601

#### 重构步骤

多个相关参数，可以考虑封装为对象传递。


