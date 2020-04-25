---
title: Hello World
date: 2015-07-01 16:49:41
tags: 生活
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

```
	$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)
<!--more-->

### Run server

```
	$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

```
	$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

```
	$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

``` java
	@Override
    public String doPostMultipartForm(String url, Map<String, Object> mapParam) {
        long start = System.currentTimeMillis();

        Response response = null;
        ResponseBody responseBody;
        String result = null;
        int code = -1;
        try {
            response = kongClient.postMultipartForm(url, mapParam);
            code = response.code();
            if (response.isSuccessful()) {
                responseBody = response.body();
                if (null != responseBody) {
                    result = responseBody.string();
                }
            }
        } catch (Exception e) {
            logger.error(ERR_LOG, url, mapParam, (System.currentTimeMillis() - start), code, e);
        } finally {
            if (null != response) {
                response.close();
            }
        }
        logger.info(LOG, url, mapParam, (System.currentTimeMillis() - start), code, result);
        resolveCode(url, mapParam, code);
        return result;
    } 

```


First Header | Second Header | Third Header
------------ | ------------- | ------------
Content Cell | Content Cell  | Content Cell
Content Cell | Content Cell  | Content Cell


First Header | Second Header | Third Header
:----------- | :-----------: | -----------:
Left         | Center        | Right
Left         | Center        | Right



这是分隔线上部分内容
---
这是分隔线上部分内容

``` graph
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->E;
    E-->F;
    D-->F;
    F-->G;
```


```graph
sequenceDiagram
    participant Alice
    participant Bob
    Alice->John: Hello John, how are you?
    loop Healthcheck
        John->John: Fight against hypochondria
    end
    Note right of John: Rational thoughts 
prevail...
    John-->Alice: Great!
    John->Bob: How about you?
    Bob-->John: Jolly good!
```


```graph
gantt
        dateFormat  YYYY-MM-DD
        title Adding GANTT diagram functionality to mermaid
        section A section
        Completed task            :done,    des1, 2014-01-06,2014-01-08
        Active task               :active,  des2, 2014-01-09, 3d
        Future task               :         des3, after des2, 5d
        Future task2               :         des4, after des3, 5d
        section Critical tasks
        Completed task in the critical line :crit, done, 2014-01-06,24h
        Implement parser and jison          :crit, done, after des1, 2d
        Create tests for parser             :crit, active, 3d
        Future task in critical line        :crit, 5d
        Create tests for renderer           :2d
        Add to mermaid                      :1d
```



