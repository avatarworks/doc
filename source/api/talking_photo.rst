活照片服务
============================

获取资源列表
---------------------
**接口描述:**
::

    获取资源列表。

**HTTP方法:**
::

    GET

**请求URL:**
::

   http(s)://api.avatarworks.com/service/v1/talkingphoto/resources

**请求Header:**

+---------------------+---------------------------------+
| 参数名称	          | 值                              |
+---------------------+---------------------------------+
| Authorization       |签名认证串,详见: :ref:`授权认证` |
+---------------------+---------------------------------+


**请求参数:**

+------------------------+------------+---------+------------------------------------------+
| 变量名                 | 格式       | 必选    | 说明                                     |
+------------------------+------------+---------+------------------------------------------+
| res_type               |   string   | false   | 值为：has_audio 或non_audio，            |
|                        |            |         | has_audio=>带音频资源，non_audio =>      |
|                        |            |         | 不带音频资源，需要用户上传音频           |
+------------------------+------------+---------+------------------------------------------+


**调用样例:**

::

    curl -X GET \
        http://api.avatarworks.com/service/v1/talkingphoto/resources \
        -H 'Authorization: AW 1bf2dac253dea653b9af3a41f7816dd5aeef5c0c:MTU1Mzg0MDYxNDrI0UoB6Nghb9AvxaNVlImCTLRyNPQAsHJji3u8xWa/vw==' \
        -H 'Content-Type: application/x-www-form-urlencoded'


**返回示例:**

::

    {
        "err_code": 0,
        "ret": [
            {
                "subtitle": "快把那红包，交出来！ 放学别走，交出红包，不到一块！快把那红包，交出来！ 放学别走，交出红包，多谢老板",
                "name": "交出红包",
                "duration": 12,
                "id": "4"
            }
        ]
    }



**PHP 代码示例:**


**Java 代码示例:**


**golang 代码示例:**


人脸重建
---------------------
**接口描述:**
::

    上传图片获取人脸重建资源。

**HTTP方法:**
::

    POST   multipart/form-data

**请求URL:**
::

   http(s)://api.avatarworks.com/service/v1/talkingphoto/face_recon

**请求Header:**

+---------------------+---------------------------------+
| 参数名称	          | 值                              |
+---------------------+---------------------------------+
| Authorization       |签名认证串,详见: :ref:`授权认证` |
+---------------------+---------------------------------+


**请求参数:**

+------------------------+------------+---------+------------------------------------------+
| 变量名                 | 格式       | 必选    | 说明                                     |
+------------------------+------------+---------+------------------------------------------+
| photo                  |   binary   | true    | 图片                                     |
+------------------------+------------+---------+------------------------------------------+


**调用样例:**

::

    curl -X POST \
        http://api.avatarworks.com/service/v1/talkingphoto/face_recon \
        -H 'Authorization: AW 1bf2dac253dea653b9af3a41f7816dd5aeef5c0c:MTU1Mzg0MDYxNDrI0UoB6Nghb9AvxaNVlImCTLRyNPQAsHJji3u8xWa/vw==' \
        -H 'Content-Type: application/x-www-form-urlencoded' \
        -H 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \
        -F 'photo=@/home/jun54555/Downloads/63c5f4e818817caf371fde9d1daa1c05 (1).jpg'



**返回示例:**

::

    {
        "err_code": 0,
        "ret": {
            "quat2": 0.0042569483630359,
            "node_scale_z": 160.72564697266,
            "quat1": -0.028118029236794,
            "texture": "face_recon\/63c5f4e818817caf371fde9d1daa1c05_texture.png",
            "x_trans": -3.4974138736725,
            "y_trans": -9.2848539352417,
            "image": "face_recon\/63c5f4e818817caf371fde9d1daa1c05_image.png",
            "quat0": 0.06465395539999,
            "quat3": 0.99750244617462,
            "obj": "face_recon\/63c5f4e818817caf371fde9d1daa1c05.obj",
            "node_scale_y": 160.72549438477,
            "node_scale_x": 160.72579956055
        }
    }




**PHP 代码示例:**


**Java 代码示例:**


**golang 代码示例:**


生成视频
---------------------
**接口描述:**
::

    上传音频文件和人脸重建信息生成视频。

**HTTP方法:**
::

    POST   multipart/form-data

**请求URL:**
::

   http(s)://api.avatarworks.com/service/v1/talkingphoto/gen_video

**请求Header:**

+---------------------+---------------------------------+
| 参数名称	          | 值                              |
+---------------------+---------------------------------+
| Authorization       |签名认证串,详见: :ref:`授权认证` |
+---------------------+---------------------------------+


**请求参数:**

+------------------------+------------+---------+------------------------------------------+
| 变量名                 | 格式       | 必选    | 说明                                     |
+------------------------+------------+---------+------------------------------------------+
| recon_json             |   string   | true    | 人脸重建接口返回的json中ret值            |
+------------------------+------------+---------+------------------------------------------+
| res_id                 |   string   | true    | 资源接口返回列表中的ID                   |
+------------------------+------------+---------+------------------------------------------+
| audio                  |   binary   | false   | 上传的mp3音频文件，mp3格式是单通道，     |
|                        |            |         | stream通道中无video stream，             |
|                        |            |         | 采样率为:16k, 16bit                      |
+------------------------+------------+---------+------------------------------------------+
| subtitle               |   string   | false   | [{"time": 1.2,  "text" : "hello"},       |
|                        |            |         | {"time": 2.2,   "text" : "world"},       |
|                        |            |         | {"time": 4.2,   "text" : ""},            |
|                        |            |         | {"time": 6.0,   "text" : "END"}]         |
|                        |            |         | 每条字幕的保留时间为下一条记录与上一条   |
|                        |            |         | 记录的时间差                             |
+------------------------+------------+---------+------------------------------------------+

**调用样例:**

::

    curl -X POST \
        http://api.avatarworks.com/service/v1/talkingphoto/gen_video \
        -H 'Authorization: AW 1bf2dac253dea653b9af3a41f7816dd5aeef5c0c:MTU2NjQ3MzAyNjoMUtsgdpDabFysh3ABY3BKkmS+DCxuAx9mgMglB4/cCA==' \
        -H 'content-type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' \
        -F audio=@/home/jun54555/Downloads/TP_m042.mp3 \
        -F 'recon_json={
              "quat2": 0.0042569483630319,
              "node_scale_z": 160.72564697266,
              "quat1": -0.028118029236794,
              "texture": "face_recon\/63c5f4e818817caf371fde9d1daa1c05_texture.png",
              "x_trans": -3.4974138736725,
              "y_trans": -9.2848539352417,
              "image": "face_recon\/63c5f4e818817caf371fde9d1daa1c05_image.png",
              "quat0": 0.06465395539999,
              "quat3": 0.99750244617462,
              "obj": "face_recon\/63c5f4e818817caf371fde9d1daa1c05.obj",
              "node_scale_y": 160.72549438477,
              "node_scale_x": 160.72579956055
          }' \
        -F res_id=4 \
        -F 'subtitle=[
              {"time": 1.2,  "text" : "hello"},
              {"time": 2.2,  "text" : "world"},
              {"time": 4.2,  "text" : ""},
              {"time": 6.0,  "text" : "END"}
          ]'




**返回示例:**

::

    {
        "err_code": 0,
        "ret": {
            "video_url": "http://tp.img.avatarworks.com/faa0c6a743fffce2a7028df22026115e.mp4",
            "cover_url": "http://tp.img.avatarworks.com/faa0c6a743fffce2a7028df22026115e.jpg"
        }
    }





**PHP 代码示例:**


**Java 代码示例:**


**golang 代码示例:**