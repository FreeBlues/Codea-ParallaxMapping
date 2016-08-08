#	Parallax Mapping on Codea(OpenGL ES 2.0) 

These days I am trying  `Parallax Mapping` and `Displacement Mapping`on Codea with my iPad2.

For this goal, I searched and read some posts. Yesterday, I got the `Parallax Mapping`, here is a [video](http://v.youku.com/v_show/id_XMTY3NTQ1MDc0OA==.html):

<embed src="http://player.youku.com/player.php/sid/XMTY3NTQ1MDc0OA==/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>

you can adjust these parameters:

-   lightPos
- 	viewPos
-  	height_scale

Using different parameters:

-	height_scale = -0.015

-	height_scale = -0.055

-	height_scale = -0.095

Because the `OpenGL ES 2.0/3.0` do not support the `Tessellation`, so someone provide another way by doing texture lookup operations in a vertex shader and calculate the displacement. But in my iPad2 without the supporting of `OpenGL ES 3.0`, so I have not verify it. Jim has done it on his mobile device(with supporting of `OpenGL ES 3.0`). Below is the screenshot:

The origin image:

![](https://chengkehan.github.io/DisplacementMapping/7.jpg)

Using displacement:

![](https://chengkehan.github.io/DisplacementMapping/1.jpg)

![](https://chengkehan.github.io/DisplacementMapping/1.gif)

Maybey It is time to buy a new iPad! :)

below is the code:

```
```

##	Reference

[38 视差贴图](http://bullteacher.com/38-parallax-mapping.html)  
[视差贴图（Parallax Mapping）与陡峭视差贴图(Steep Palallax Mapping)](http://blog.csdn.net/xiaoge132/article/details/51173002)  
[Parallax Occlusion Mapping in GLSL](http://sunandblackcat.com/tipFullView.php?topicid=28)
[Jim's GameDev Blog: 置换贴图 Displacement Mapping](https://chengkehan.github.io/DisplacementMapping.html)
