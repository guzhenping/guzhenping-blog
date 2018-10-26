
## 前言
Flink在实时流处理领域越来越热，以阿里为首的企业正在投入更多的资源。在实际工作中也遇到了流处理的场景，特此学习一下。

## Demo制作
先看一个demo:

WordCount.java

```java
package com.gzp.batch;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.utils.ParameterTool;
import org.apache.flink.util.Collector;

public class WordCount {
    public static void main(String[] args) throws Exception {
        final ParameterTool params = ParameterTool.fromArgs(args);
        final ExecutionEnvironment env = ExecutionEnvironment.createCollectionsEnvironment();
        DataSet<String> text = WordCountData.getDefaultTextLineDataSet(env);
        DataSet<Tuple2<String, Integer>> counts =
                // split up the lines in pairs (2-tuples) containing: (word,1)
                text.flatMap(new Tokenizer())
                    .groupBy(0)
                    .sum(1);
        counts.print();
    }
    public static final class Tokenizer implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
            // emit the pairs
            for (String token : value.toLowerCase().split("\\W+")) {
                if (token.length() > 0) {
                    out.collect(new Tuple2<>(token, 1));
                }
            }
        }
    }
}
```

WordCountData.java

```
package com.gzp.batch;

import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;

public class WordCountData {
    public static final String[] WORDS = new String[] {
            "To be, or not to be,--that is the question:--",
            "Whether 'tis nobler in the mind to suffer",
            "The slings and arrows of outrageous fortune",
            "Or to take arms against a sea of troubles,",
            "And by opposing end them?--To die,--to sleep,--",
            "No more; and by a sleep to say we end",
            "The heartache, and the thousand natural shocks",
            "That flesh is heir to,--'tis a consummation",
            "Devoutly to be wish'd. To die,--to sleep;--",
            "To sleep! perchance to dream:--ay, there's the rub;",
            "For in that sleep of death what dreams may come,",
            "When we have shuffled off this mortal coil,",
            "Must give us pause: there's the respect",
            "That makes calamity of so long life;",
            "For who would bear the whips and scorns of time,",
            "The oppressor's wrong, the proud man's contumely,",
            "The pangs of despis'd love, the law's delay,",
            "The insolence of office, and the spurns",
            "That patient merit of the unworthy takes,",
            "When he himself might his quietus make",
            "With a bare bodkin? who would these fardels bear,",
            "To grunt and sweat under a weary life,",
            "But that the dread of something after death,--",
            "The undiscover'd country, from whose bourn",
            "No traveller returns,--puzzles the will,",
            "And makes us rather bear those ills we have",
            "Than fly to others that we know not of?",
            "Thus conscience does make cowards of us all;",
            "And thus the native hue of resolution",
            "Is sicklied o'er with the pale cast of thought;",
            "And enterprises of great pith and moment,",
            "With this regard, their currents turn awry,",
            "And lose the name of action.--Soft you now!",
            "The fair Ophelia!--Nymph, in thy orisons",
            "Be all my sins remember'd."
    };
    public static DataSet<String> getDefaultTextLineDataSet(ExecutionEnvironment env) {
        return env.fromElements(WORDS);
    }
}
```
上述代码核心是将一段文本按单词切割，统计词频。flatMap()调用udf将文本切割并生成结构化数据，按单词分组后再sum。

不得不说，java写的flink task代码太长....

## Core Concepts

## Flink Case

####

## 参考资料、

- 核心概念翻译

[Flink架构、原理与部署测试详解](https://blog.csdn.net/xiaoping2017/article/details/79846158)

- 简单的Scala Demo应用： 

[flink demo](https://github.com/liguohua-bigdata/simple-flink/blob/master/book/stream/customSource/customSourceScala.md)

- 关于动态表: 

[在数据流中使用SQL查询：Apache Flink中的动态表的持续查询](http://www.10tiao.com/html/157/201707/2653162664/1.html)
