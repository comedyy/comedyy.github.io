# c#中的CultureInfo.

1. 遇到一个bug. float.Parse("3.14") 会报错。是的，在法语的机器上。 为啥会出现这样情况，原因是float.Parse()其实有个默认参数 CultureInfo的，如果它是法语，他会根据法语的规则去解析。而法语的3.14是写成3,14. 所以就出问题了。如果使用 CultureInfo.InvariantCulture,就是默认方式处理，float.Parse("3.14")就可以直接输出3.14.
同样，如果你使用 3.14.ToString(new CultureInfo("fr-FR")) 同样会输出"3,14"