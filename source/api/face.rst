人脸服务
============================

人脸重建
---------------------

**接口描述:**
::

    上传图片生成用于人脸重建的3D资源

**HTTP方法:**
::

    POST

**请求URL:**
::

    http(s)://api.avatarworks.com/service/v2/face/recon

**请求Header:**

+---------------------+---------------------------------+
| 参数名称	          | 值                              |
+---------------------+---------------------------------+
| Authorization       |签名认证串,详见: :ref:`授权认证` |
+---------------------+---------------------------------+


**请求参数:**

+------------------------+------------+------------------------------------------+
| 变量名                 | 格式       | 说明                                     |
+------------------------+------------+------------------------------------------+
| photo                  |   binary   |上传的图片内容,以multipart-form的形式上传 |
+------------------------+------------+------------------------------------------+


**调用样例:**

::

    curl -X POST \
        http://api.avatarworks.com/service/v1/face/recon \
        -H 'Authorization: AW 1bf2dac253dea653b9af3a41f7816dd5aeef5c0c:MTU1Mzg0MDYxNDrI0UoB6Nghb9AvxaNVlImCTLRyNPQAsHJji3u8xWa/vw==' \
        -H 'Content-Type: application/x-www-form-urlencoded' \
        -H 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \
        -F 'photo=@/home/test/63c5f4e818817caf371fde9d1daa1c05.jpg'



**返回示例:**

::

    {
        "err_code": 0,
        "ret": {

            "target_data" : "{base64数据}",
            "mapping"     : "{base64数据}",
            "face_credibility_level": 0,
            "face_width": 642
        }
    }

**PHP 代码示例:**


**Java 代码示例:**


**golang 代码示例:**
