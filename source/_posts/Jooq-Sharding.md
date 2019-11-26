---
title: Jooq 分表
date: 2019-11-19 15:41:31
tags:
  - Java
  - Jooq
categories: code
---

# Code

```Java
import com.google.common.cache.CacheBuilder;
import com.google.common.cache.CacheLoader;
import com.google.common.cache.LoadingCache;
import org.jooq.Configuration;
import org.jooq.conf.MappedSchema;
import org.jooq.conf.MappedTable;
import org.jooq.conf.RenderMapping;
import org.jooq.conf.Settings;
import org.jooq.impl.TableImpl;

import java.util.function.BiFunction;
import java.util.function.Function;
import java.util.regex.Pattern;

/**
 * @param <R> row
 * @param <S> sharding value
 */
public class ConfigurationHolder<R, S> {

    private final LoadingCache<S, Configuration> cache;
    private final Function<R, S> shardingFn;

    /**
     * @param shardingFn (shardingParameter) -> shardingValue
     * @param renameFn   (originalTableName, shardingValue) -> shardedTableName
     */
    public static <R, S> ConfigurationHolder<R, S> newInstance(Configuration configuration,
                                                               TableImpl table,
                                                               Function<R, S> shardingFn,
                                                               BiFunction<String, S, String> renameFn) {
        return new ConfigurationHolder<>(configuration, table.getName(), shardingFn, renameFn);
    }

    private ConfigurationHolder(Configuration configuration,
                                String originalName,
                                Function<R, S> shardingFn,
                                BiFunction<String, S, String> renameFn) {
        this.shardingFn = shardingFn;
        this.cache = CacheBuilder
          .newBuilder()
          .build(new CacheLoader<S, Configuration>() {
              @Override
              public Configuration load(S s) {
                  Settings settings = new Settings()
                    .withRenderMapping(new RenderMapping()
                      .withSchemata(new MappedSchema()
                        .withInputExpression(Pattern.compile(".*"))
                        .withTables(new MappedTable()
                          .withInput(originalName)
                          .withOutput(renameFn.apply(originalName, s)))));
                  return configuration.derive(settings);
              }
          });
    }

    public Configuration get(R shardingParameter) {
        return cache.getUnchecked(shardingFn.apply(shardingParameter));
    }
}
```

# Use

```Java
DefaultConfiguration config;
TableImpl<Record> fakeTable;

ConfigurationHolder<Integer, Integer> holder = ConfigurationHolder.newInstance(config, fakeTable, id -> id & i & ((1 << 4) - 1), (originalName, shardingVal) -> originalName + shardingVal);

int id = 1000;
DSL.using(holder.get(id))
  .selectFrom(table);
```

# 解释

最初试想 `jooq` 的 `SQL` 语句来源于对象, `TableImpl` 对象的构造函数中有一个包含名称的构造函数.

> ```Java
> public TableImpl(String name) {
>   this(DSL.name(name));
> }
> ```

似乎是这个决定了表名. 那么创建一个修改过表名的 `TableImpl` 对象应该能实现表名的修改. 结果当然未尝所愿.

这个构造器只是用来 *alias* (取一个别名).

关于表的重命名, 官网上给出的做法是

> ```Java
> Settings settings = new Settings()
>   .withRenderMapping(new RenderMapping()
>     .withSchemata(new MappedSchema()
>       .withInput("DEV")
>       .withTables(new MappedTable()
>         .withInput("AUTHOR")
>         .withOutput("MY_APP__AUTHOR"))));
> ```

修改 `Settings`, 指定相应的 `Schema` 的相应的 `Table` 进行重命名.

于是, 就有了[上述](#Code)代码. 对 `Configuration` 进行 `derive` 且缓存. 使用时, 从 `LoadingCache` 中获取派生的 `Configuration` , 使用 `DSL.using(Configuration configuration)` 进行 `SQL` 操作.

# 总结

尽整些没屁用的, 好好的数据库分表不用, 非要折腾个 mybatis 插件分表.

# 后续

> ```Java
> public static DSLContext using(Configuration configuration) {
>   return new DefaultDSLContext(configuration);
> }
> ```

`DSLContext.using` 是创建一个新的 `DefaultDSLContext`, 其实可以把 `DSLContext` 给缓存起来, 而不仅仅是 `Configuration`.