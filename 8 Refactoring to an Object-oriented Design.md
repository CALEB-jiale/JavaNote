# Refactoring to an Object-oriented Design

## 提取“类”

1. 选中要提取的方法名
2. 在上方菜单栏找到 Refactor 并选择 Refactor This，快捷键 control + T
3. 在弹出的选项栏选择 Move，快捷键 F6
4. 在弹出的菜单栏选择移动的目的地址
5. 在 Member 中选择要被移动的项
6. 在 Visibility 中选择对应的选项，一般选择 Public
7. 点击 Refactor

## Constructor

1. 在上方菜单栏找到 Code 并选择 Generate，快捷键 command + N
2. 在弹出的选项栏选择 Constructor
3. 在弹出的菜单栏中选择要初始化的字段

## 删除重复参数

1. 选中要提取的方法名
2. 在上方菜单栏找到 Refactor 并选择 Refactor This，快捷键 control + T
3. 在弹出的选项栏选择 Change Signature，快捷键 command + F6
4. 在弹出的菜单栏中选择要删除的参数
5. 点击下方的减号将其删除
6. 点击 Refactor

## Find Usages

1. 选中“类”后右击
2. 选择 Find Usages， 快捷键 option + F7

## Extract a field

1. 在上方菜单栏找到 Refactor 并选择 Refactor This，快捷键 control + T
2. 在弹出的选项栏选择 Introduce Field，快捷键 command + option + F

## Convert to an instance method

1. 在上方菜单栏找到 Refactor 并选择 Refactor This，快捷键 control + T
2. 在弹出的选项栏选择 Convert To Instance Method

## Inline variables

作用：取消多余的变量声明

1. 在上方菜单栏找到 Refactor 并选择 Inline Variables，快捷键 command + option + N