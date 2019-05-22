# SPEL表达式的使用

## 解析过程

使用 ExpressionParser 基于 ParserContext 将字符串解析为 Expression，Expression 再根据 EvaluationContext 计算表达式的值。

## note 1:

在单独使用SpEL时需要创建解析器、解析表达式、以及求值上下文和对应的根对象。但是在实际使用过程中、更常用的使用方式是只需要在配置文件里面配置SpEL字符串表达式即可，例如针对Spring Bean或者Spring Web Flow的定义。在这种场景下解析器，求值上下文，根对象和任何事先定义的变量都会被容器默认创建好，用户除了写表达式不需要做任何其他事情。

## note 2:

StandardEvaluationContext创建相对比较耗资源，在重复多次使用的场景下内部会缓存部分中间状态加快后续的表达式求值效率。因此建议在使用过程中尽可能被缓存和重用，而不是每次在表达式求值时都重新创建一个对象。



## 解释器

private ExpressionParser expressionParser;

## 缓存 解析表达式

private final Map<String, Expression> expressionCache = new ConcurrentHashMap<>(256);

## 缓存 表达式解析上下文

private final Map<BeanExpressionContext, StandardEvaluationContext> evaluationCache = new ConcurrentHashMap<>(8);