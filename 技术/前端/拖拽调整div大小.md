![690ae76d3dc920949ab4c2f7301b3121.gif](https://raw.githubusercontent.com/mervynlam/Pictures/master/20200821112209.gif)

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <style type="text/css">
        div {
            height: 200px;
            position: absolute;
            top: 0px;
        }
        .right {
            right: 0px;
        }
        .left {
            left: 0px;
        }
        .right,.left {
            width: 49.75%;
        }
        .drag {
            width: 0.5%;
            left: 49.75%
        }
    </style>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.1/jquery.js"></script>
</head>
<body>
<div class="left" style="background-color: grey"></div>
<div class="drag"></div>
<div class="right" style="background-color: cadetblue"></div>
<script type="text/javascript">
    //调整大小
    var resizeDiv = {
        init: function () {
            resizeFlag = false;
            this.mouseDown();
            this.mouseMove();
            this.mouseUp();
            this.mouseLeave();
        },
        mouseDown: function () {
            $(".drag").on("mousedown", function () {
            	//按下设为true，允许根据鼠标位置调整大小
                resizeFlag = true;
            });
        },
        mouseMove: function () {
            $(document).on("mousemove", function (e) {
        		//resizeFlag为true时才允许根据鼠标位置调整大小
                if (resizeFlag) {
                    var mousePointX = e.pageX;
                    if (mousePointX < $(window).width()) {
                        $(".left").css("width", mousePointX);
                        $(".right").css("width", $(window).width() - $(".left").width() - $(".drag").width());
                    }
                }
            });
        },
        mouseLeave: function () {
            $(document).on("mouseleave", function (e) {
            	//鼠标离开，标识设为false，不再根据鼠标位置调整大小
                resizeFlag = false;
            });
        },
        mouseUp: function () {
            $(document).on("mouseup", function () {
            	//鼠标抬起，标识设为false，不再根据鼠标位置调整大小
                resizeFlag = false;
            });
        }
    };
    resizeDiv.init();
</script>
</body>
</html>
```