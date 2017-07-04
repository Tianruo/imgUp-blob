# imgUp-blob
图片预先显示，并转为二进制流，上传到ajax


//点击上传图片，之后，创建oo临时对象，push进对应的list数组



    function imgUp(tt, ii) {
        if($(tt).val()!=''){
            if ($(tt).parent().parent().find('.preview').length < 3) {
                var $file = $(tt);
                var fileObj = $file[0];
                var windowURL = window.URL || window.webkitURL;
                var dataURL;
                $file.parent().parent().append('<img class="preview">');
                var $img = $file.parent().parent().find('.preview:last');
                var _name = $(tt)[0].files[0].name;
                var oo = {};
                oo.fileName = _name;


                if (fileObj && fileObj.files && fileObj.files[0]) {
                    dataURL = windowURL.createObjectURL(fileObj.files[0]);//核心：根据文件创造一个URL，blob对象，先显出来
                    getBase64(dataURL)//传入blob的url，可以得到真正的图片二进制流码
                        .then(function (base64) {
                            oo.file = base64;
                            imgAll['list' + ii].push(oo);//ok，保存到imgAll对象里，等着ajax上传即可
                        }, function (err) {
                            console.log(err);
                        });
                    $img.attr('src', dataURL);
                } else {
                    dataURL = $file.val();
                    var imgObj = document.querySelector(".preview");
                    // 两个坑:
                    // 1、在设置filter属性时，元素必须已经存在在DOM树中，动态创建的Node，也需要在设置属性前加入到DOM中，
                    先设置属性在加入，无效；
                    // 2、src属性需要像下面的方式添加，上面的两种方式添加，无效；
                    imgObj.style.filter = "progid:DXImageTransform.Microsoft.AlphaImageLoader(sizingMethod=scale)";
                    imgObj.filters.item("DXImageTransform.Microsoft.AlphaImageLoader").src = dataURL;
                }
            } else {
                layer.alert('最多只能上传3张图片哦', {icon: 6});
            }
            $(tt).parent().parent().find('.eva_con4 span').html($(tt).parent().parent().find('img').length-2+'/3');
        }

    }
    //传入图片路径，返回base64（修改：去除前缀）
    function getBase64(img) {
        function getBase64Image(img, width, height) {//width、height调用时传入具体像素值，控制大小 ,不传则默认图像大小
            var canvas = document.createElement("canvas");
            canvas.width = width ? width : img.width;
            canvas.height = height ? height : img.height;

            var ctx = canvas.getContext("2d");
            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
            var dataURL2 = canvas.toDataURL();//核心：canvas.toDataURL()把canvas图片转为二进制流码
            var index = dataURL2.indexOf('base64') + 7;
            var dataURL = dataURL2.slice(index);
            return dataURL;
        }

        var image = new Image();
        image.src = img;
        var deferred = $.Deferred();
        if (img) {
            image.onload = function () {
                deferred.resolve(getBase64Image(image));//将base64传给done上传处理
            }
            return deferred.promise();//问题要让onload完成后再return sessionStorage['imgTest']
        }
    }
