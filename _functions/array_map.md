---
title: array_map - Syntax, Performance
description: Introduction of php function array_map in terms of it syntax and performance in compare with various php versions as well as other functions. 
tags: array, callback, syntax, performance, php, hhvm
layout: article
---

## Syntax
{% highlight php startinline=true %}
    
    array array_map(callable $callback, array $array1 [, array $arrayn])

{% endhighlight %}

## Performance

- 10000x foreach take 1.14517 seconds
- 10000x array_map take 1.66716 seconds
  
 array_map() is slower then conventional foreach() loop with a 1.13 factor


