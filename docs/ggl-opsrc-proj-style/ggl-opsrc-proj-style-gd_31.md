# 环境

## STDOUT vs STDERR

Tip

所有的错误信息都应该被导向 STDERR。

这使得从实际问题中分离出正常状态变得更容易。

推荐使用类似如下函数，将错误信息和其他状态信息一起打印出来。

```
err() {
    echo "[$(date +'%Y-%m-%dT%H:%M:%S%z')]: $@" >&2
}

if ! do_something; then
    err "Unable to do_something"
    exit "${E_DID_NOTHING}"
fi 
```

© Copyright . Created using [Sphinx](http://sphinx-doc.org/) 1.3.5.