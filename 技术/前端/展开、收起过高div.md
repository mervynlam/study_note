![d29308557297cb9cc3637fb62976c858.gif](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071626527.gif)

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <style type="text/css">
        .contentDiv {
            width: 300px;
        }
        .hideDiv {
            display: -webkit-box;
            -webkit-box-orient: vertical;
            -webkit-line-clamp: 3;
            overflow: hidden;
        }
    </style>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.1/jquery.js"></script>
</head>
<body>
<div>
    <div class="contentDiv hideDiv">
        Kubernetes一个核心的特点就是能够自主的管理容器来保证云平台中的容器按照用户的期望状态运行着（比如用户想让apache一直运行，用户不需要关心怎么去做，Kubernetes会自动去监控，然后去重启，新建，总之，让apache一直提供服务），管理员可以加载一个微型服务，让规划器来找到合适的位置，同时，Kubernetes也系统提升工具以及人性化方面，让用户能够方便的部署自己的应用（就像canary deployments）。
    </div>
    <a href="javascript:;" class="showHideA">[展开全部]</a>
</div>
<script type="text/javascript">
    $(".showHideA").on("click", function() {
        var contentDiv = $(".contentDiv");
        if (contentDiv.hasClass("hideDiv")) {
            contentDiv.removeClass("hideDiv");
            $(this).text("[收起]");
        } else {
            contentDiv.addClass("hideDiv");
            $(this).text("[展开全部]");
        }
    });

</script>
</body>
</html>
```