---
title: Spring 表达式语言 (SpEL)
date: 2022-06-04 23:41
tags: 
  - Spring
  - SpEL
categories:
  - [Spring]
---


# Spring 表达式语言 (SpEL)

## 使用 Spring 的表达式接口进行表达式求值
下面的代码引入了 SpEL API 来评估文字字符串表达式 'Hello World'
```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'");
String message = (String) exp.getValue();
```
常用的 SpEL 类和接口位于包org.springframework.expression 及其子包和spel.support中。

作为方法调用的示例，我们在字符串文字上调用“concat”方法
```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')");
String message = (String) exp.getValue();
```
> 这将返回 'Hello World!'

使用点号来获取属性
```
ExpressionParser parser = new SpelExpressionParser();
// invokes 'getBytes().length'
Expression exp = parser.parseExpression("'Hello World'.bytes.length");  
int length = (Integer) exp.getValue();
```
调用方法
```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("new String('hello world').toUpperCase()");
String message = exp.getValue(String.class);
```
### EvaluationContext接口
EvaluationContext在评估表达式以解析属性、方法、字段并帮助执行类型转换时使用该接口。开箱即用的实现使用反射来StandardEvaluationContext操作对象，缓存 j ava.lang.reflect的Method和 实例以提高性能。
```
class Simple {
    public List<Boolean> booleanList = new ArrayList<Boolean>();
}
    	
Simple simple = new Simple();

simple.booleanList.add(true);

StandardEvaluationContext simpleContext = new StandardEvaluationContext(simple);

// false is passed in here as a string.  SpEL and the conversion service will 
// correctly recognize that it needs to be a Boolean and convert it
parser.parseExpression("booleanList[0]").setValue(simpleContext, "false");

// b will be false
Boolean b = simple.booleanList.get(0);
```


 
## 语法参考
https://docs.spring.io/spring-framework/docs/3.0.0.M4/reference/html/ch06s05.html
 
### 文字表达式
支持的文字表达式类型有字符串、日期、数值（整数、实数和十六进制）、布尔值和空值。字符串由单引号分隔。
```
ExpressionParser parser = new SpelExpressionParser();

// evals to "Hello World"
String helloWorld = (String) parser.parseExpression("'Hello World'").getValue(); 

double avogadrosNumber  = (Double) parser.parseExpression("6.0221415E+23").getValue();  

// evals to 2147483647
int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();  

boolean trueValue = (Boolean) parser.parseExpression("true").getValue();

Object nullValue = parser.parseExpression("null").getValue();
 ```
数字支持使用负号、指数记数法和小数点。默认情况下，使用 Double.parseDouble() 解析实数。

### 属性、数组、Lists、Maps、Indexers
使用**点**来表示属性引用
```
int year = (Integer) parser.parseExpression("Birthdate.Year + 1900").getValue(context); 

String city = (String) parser.parseExpression("placeOfBirth.City").getValue(context);
```
数组和列表的内容是使用**方括号**表示
```
ExpressionParser parser = new SpelExpressionParser();

// Inventions Array
StandardEvaluationContext teslaContext = new StandardEvaluationContext(tesla);

// evaluates to "Induction motor"
String invention = parser.parseExpression("inventions[3]").getValue(teslaContext, 
                                                                    String.class); 

// Members List
StandardEvaluationContext societyContext = new StandardEvaluationContext(ieee);

// evaluates to "Nikola Tesla"
String name = parser.parseExpression("Members[0].Name").getValue(societyContext, String.class);

// List and Array navigation
// evaluates to "Wireless communication"
String invention = parser.parseExpression("Members[0].Inventions[6]").getValue(societyContext,
                                                                               String.class);
```
map的内容是通过在括号内指定文字键值来获得的
```
// Officer's Dictionary

Inventor pupin = parser.parseExpression("Officers['president']").getValue(societyContext, 
                                                                          Inventor.class);

// evaluates to "Idvor"
String city = 
    parser.parseExpression("Officers['president'].PlaceOfBirth.City").getValue(societyContext,
                                                                               String.class);

// setting values
parser.parseExpression("Officers['advisors'][0].PlaceOfBirth.Country").setValue(societyContext,
                                                                                "Croatia");

```

### 方法
使用典型的 Java 编程语法调用方法。还可以在文字上调用方法，支持方法参数
```
// string literal, evaluates to "bc"
String c = parser.parseExpression("'abc'.substring(2, 3)").getValue(String.class);

// evaluates to true
boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')").getValue(societyContext,
                                                                                Boolean.class);
```

### 运算符号
#### 关系运算符
使用标准运算符表示法支持等于、不等于、小于、小于或等于、大于和大于或等于。
```
// 计算结果为真
boolean trueValue = parser.parseExpression( " 2 == 2" ).getValue(Boolean.class );

// 计算结果为假
boolean falseValue = parser.parseExpression( " 2 < -5.0" ).getValue(Boolean.class );

// 计算结果为真
boolean trueValue = parser.parseExpression( " 'black' < 'block'" ).getValue(Boolean.class );
```
除了标准的关系运算符之外，SpEL 还支持“instanceof”和基于正则表达式的“匹配”运算符。
```
// evaluates to false
boolean falseValue = parser.parseExpression("'xyz' instanceof T(int)").getValue(Boolean.class);

// evaluates to true
boolean trueValue = 
     parser.parseExpression("'5.00' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);

//evaluates to false
boolean falseValue = 
     parser.parseExpression("'5.0067' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);
```
#### 逻辑运算符
支持的逻辑运算符有 and、or 和 not。
```
// -- AND --

// evaluates to false
boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);

// evaluates to true
String expression =  "isMember('Nikola Tesla') and isMember('Mihajlo Pupin')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- OR --

// evaluates to true
boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);

// evaluates to true
String expression =  "isMember('Nikola Tesla') or isMember('Albert Einstien')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- NOT --

// evaluates to false
boolean falseValue = parser.parseExpression("!true").getValue(Boolean.class);


// -- AND and NOT --
String expression =  "isMember('Nikola Tesla') and !isMember('Mihajlo Pupin')";
boolean falseValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);
```

#### 数学运算符
加法运算符可用于数字、字符串和日期。减法可用于数字和日期。乘法和除法只能用于数字。其他支持的数学运算符是模数 (%) 和指数幂 (^)。执行标准运算符优先级。这些运算符如下所示
```
// Addition
int two = parser.parseExpression("1 + 1").getValue(Integer.class); // 2

String testString = 
   parser.parseExpression("'test' + ' ' + 'string'").getValue(String.class);  // 'test string'

// Subtraction
int four =  parser.parseExpression("1 - -3").getValue(Integer.class); // 4

double d = parser.parseExpression("1000.00 - 1e4").getValue(Double.class); // -9000

// Multiplication
int six =  parser.parseExpression("-2 * -3").getValue(Integer.class); // 6

double twentyFour = parser.parseExpression("2.0 * 3e0 * 4").getValue(Double.class); // 24.0

// Division
int minusTwo =  parser.parseExpression("6 / -3").getValue(Integer.class); // -2

double one = parser.parseExpression("8.0 / 4e0 / 2").getValue(Double.class); // 1.0

// Modulus
int three =  parser.parseExpression("7 % 4").getValue(Integer.class); // 3

int one = parser.parseExpression("8 / 5 % 2").getValue(Integer.class); // 1

// Operator precedence
int minusTwentyOne = parser.parseExpression("1+2-3*8").getValue(Integer.class); // -21
```

### 赋值

### 类型
### 构造器

###  变量 
可以使用语法#variableName 在表达式中引用变量。使用 StandardEvaluationContext 上的 setVariable 方法设置变量。
```
Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
StandardEvaluationContext context = new StandardEvaluationContext(tesla);
context.setVariable("newName", "Mike Tesla");

parser.parseExpression("Name = #newName").getValue(context);

System.out.println(tesla.getName()) // "Mike Tesla"
```
#### #this 变量
变量 #this 始终被定义并引用当前评估对象

### 函数

### 三元运算符（If-Then-Else）



