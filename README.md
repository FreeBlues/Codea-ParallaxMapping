#	Parallax Mapping on Codea(OpenGL ES 2.0) 

##	Parallax Mapping

These days I am trying  `Parallax Mapping` and `Displacement Mapping` on Codea with my iPad2.

For this goal, I searched and read some posts. Yesterday, I got the `Parallax Mapping` successful on my iPad2, here is a [video](http://v.youku.com/v_show/id_XMTY3NTQ1MDc0OA==.html):

<embed src="http://player.youku.com/player.php/sid/XMTY3NTQ1MDc0OA==/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>

You can adjust these parameters to get different results:

-	lightPos: The light position
- 	viewPos: The eye position
-  	height_scale: The scaler to set the 

Using different parameters:

-	height_scale = -0.015

![](https://github.com/FreeBlues/Codea-ParallaxMapping/blob/master/images/I015.PNG)

-	height_scale = -0.055

![](https://github.com/FreeBlues/Codea-ParallaxMapping/blob/master/images/I055.PNG)

-	height_scale = -0.095

![](https://github.com/FreeBlues/Codea-ParallaxMapping/blob/master/images/I095.PNG)

##	Displacement Mapping

Because the `OpenGL ES 2.0/3.0` do not support the `Tessellation`, so Jim provide another way by doing texture lookup operations in a vertex shader and calculate the displacement. But in my iPad2 without the supporting of `OpenGL ES 3.0`, so I can not verify it. Jim has done it on his mobile device(with supporting of `OpenGL ES 3.0`). Below are the screenshots:

The origin image:

![](https://chengkehan.github.io/DisplacementMapping/7.jpg)

Using Displacement Mapping:

![](https://chengkehan.github.io/DisplacementMapping/1.jpg)

![](https://chengkehan.github.io/DisplacementMapping/1.gif)

Maybey It is time to buy a new iPad! :)

below is the code(modify from [38 视差贴图](http://bullteacher.com/38-parallax-mapping.html)):

```

function setup()
    displayMode(OVERLAY)
    print("试验 OpenGL ES 2.0 中位移贴图的例子")
    print("Test the Parallax Mapping in OpenGL ES 2.0")

    img1 = readImage("Dropbox:dm")
    img2 = readImage("Dropbox:dnm1")
    img3 = readImage("Dropbox:dm1")
    local w,h = WIDTH,HEIGHT
    local c = color(223, 218, 218, 255)
    m3 = mesh()
    m3i = m3:addRect(w/2,h/2,w/1,h/1)
    m3:setColors(c)
    m3.texture = img1
    m3:setRectTex(m3i,0,0,1,1)
    m3.shader = shader(shaders.vs,shaders.fs)
    m3.shader.diffuseMap = img1
    m3.shader.normalMap = img2
    m3.shader.depthMap =  img3

    -- local tb = m3:buffer("tangent")
    -- tb:resize(6)
    tchx,tchy = 0,0
end

function draw()
    background(40, 40, 50)

    perspective()
    -- camera(e.x, e.y, e.z, p.x, p.y, p.z)
    -- 用于立方体
    -- camera(300,300,600, 0,500,0, 0,0,1)
    -- 用于平面位移贴图
    camera(WIDTH/2,HEIGHT/2,1000, WIDTH/2,HEIGHT/2,-200, 0,1,0)
    
    -- mySp:Sphere(100,100,100,0,0,0,10)
    light = vec3(tchx, tchy, 100.75)
    view = vec3(tchx, tchy, 300.75)
    -- light = vec3(300, 300, 500)
    -- rotate(ElapsedTime*5,0,1,0)
    -- m3:setRect(m2i,tchx,tchy,WIDTH/100,HEIGHT/100)

    setShaderParam(m3)
    m3:draw()   
end

function touched(touch)
    if touch.state == BEGAN or touch.state == MOVING then
        tchx=touch.x+10
        tchy=touch.y+10
    end
end

function setShaderParam(m)
    m.shader.model = modelMatrix()
    m.shader.lightPos = light 
    -- m.shader.lightPos = vec3(0.5, 1.0, 900.3)
    m.shader.viewPos = vec3(WIDTH/2,HEIGHT/2,5000) 
    m.shader.viewPos = view
    -- m.shader.viewPos = vec3(0.0, 0.0, 90.0)
    m.shader.parallax = true
    m.shader.height_scale = -0.015
end

-- 试验 视差贴图 中的例子
shaders = {
vs = [[
attribute vec4 position;
attribute vec3 normal;
attribute vec2 texCoord;
//attribute vec3 tangent;
//attribute vec3 bitangent;

varying vec3 vFragPos;
varying vec2 vTexCoords;
varying vec3 vTangentLightPos;
varying vec3 vTangentViewPos;
varying vec3 vTangentFragPos;

uniform mat4 projection;
uniform mat4 view;
uniform mat4 model;
uniform mat4 modelViewProjection;

uniform vec3 lightPos;
uniform vec3 viewPos;

void main()
{
    //gl_Position = projection * view * model * position;
    gl_Position = modelViewProjection * position;
    vFragPos = vec3(model * position);   
    vTexCoords = texCoord;
    
    // 根据法线 N 计算 T，B
    vec3 tangent; 
    vec3 binormal; 

    // 使用顶点属性法线，并归一化
    vec3 Normal = normalize(normal*2.0-1.0);

    vec3 c1 = cross(Normal, vec3(0.0, 0.0, 1.0)); 
    vec3 c2 = cross(Normal, vec3(0.0, 1.0, 0.0)); 
	
    if ( length(c1) > length(c2) ) { tangent = c1;	} else { tangent = c2;}
	
    // 归一化切线和次法线
    tangent = normalize(tangent);
    binormal = normalize(cross(normal, tangent)); 
    
    vec3 T = normalize(mat3(model) * tangent);
    vec3 B = normalize(mat3(model) * binormal);
    vec3 N = normalize(mat3(model) * normal);
    mat3 TBN = mat3(T, B, N);

    vTangentLightPos = lightPos*TBN;
    vTangentViewPos  = viewPos*TBN;
    vTangentFragPos  = vFragPos*TBN;
}
]],

fs = [[
precision highp float;

varying vec3 vFragPos;
varying vec2 vTexCoords;
varying vec3 vTangentLightPos;
varying vec3 vTangentViewPos;
varying vec3 vTangentFragPos;

uniform sampler2D diffuseMap;
uniform sampler2D normalMap;
uniform sampler2D depthMap;

uniform bool parallax;
uniform float height_scale;

vec2 ParallaxMapping(vec2 texCoords, vec3 viewDir);
vec2 ParallaxMapping1(vec2 texCoords, vec3 viewDir);

// The Parallax Mapping
vec2 ParallaxMapping1(vec2 texCoords, vec3 viewDir)
{ 
    float height =  texture2D(depthMap, texCoords).r;     
    return texCoords - viewDir.xy / viewDir.z * (height * height_scale);            
}

// Parallax Occlusion Mapping
vec2 ParallaxMapping(vec2 texCoords, vec3 viewDir)
{ 
    // number of depth layers
    const float minLayers = 10.0;
    const float maxLayers = 50.0;
    float numLayers = mix(maxLayers, minLayers, abs(dot(vec3(0.0, 0.0, 1.0), viewDir)));  
    // calculate the size of each layer
    float layerDepth = 1.0 / numLayers;
    // depth of current layer
    float currentLayerDepth = 0.0;
    // the amount to shift the texture coordinates per layer (from vector P)
    vec2 P = viewDir.xy / viewDir.z * height_scale; 
    vec2 deltaTexCoords = P / numLayers;
  
    // get initial values
    vec2  currentTexCoords     = texCoords;
    float currentDepthMapValue = texture2D(depthMap, currentTexCoords).r;
      
    while(currentLayerDepth < currentDepthMapValue)
    {
        // shift texture coordinates along direction of P
        currentTexCoords -= deltaTexCoords;
        // get depthmap value at current texture coordinates
        currentDepthMapValue = texture2D(depthMap, currentTexCoords).r;  
        // get depth of next layer
        currentLayerDepth += layerDepth;  
    }
    
    // -- parallax occlusion mapping interpolation from here on
    // get texture coordinates before collision (reverse operations)
    vec2 prevTexCoords = currentTexCoords + deltaTexCoords;

    // get depth after and before collision for linear interpolation
    float afterDepth  = currentDepthMapValue - currentLayerDepth;
    float beforeDepth = texture2D(depthMap, prevTexCoords).r - currentLayerDepth + layerDepth;
 
    // interpolation of texture coordinates
    float weight = afterDepth / (afterDepth - beforeDepth);
    vec2 finalTexCoords = prevTexCoords * weight + currentTexCoords * (1.0 - weight);

    return finalTexCoords;
}

void main()
{           
    // Offset texture coordinates with Parallax Mapping
    vec3 viewDir = normalize(vTangentViewPos - vTangentFragPos);
    vec2 texCoords = vTexCoords;
    if(parallax)
        texCoords = ParallaxMapping(vTexCoords,  viewDir);
        
    // discards a fragment when sampling outside default texture region (fixes border artifacts)
    if(texCoords.x > 1.0 || texCoords.y > 1.0 || texCoords.x < 0.0 || texCoords.y < 0.0)
       discard;

    // Obtain normal from normal map
    vec3 normal = texture2D(normalMap, texCoords).rgb;
    normal = normalize(normal * 2.0 - 1.0);   
   
    // Get diffuse color
    vec3 color = texture2D(diffuseMap, texCoords).rgb;
    // Ambient
    vec3 ambient = 0.1 * color;
    // Diffuse
    vec3 lightDir = normalize(vTangentLightPos - vTangentFragPos);
    float diff = max(dot(lightDir, normal), 0.0);
    vec3 diffuse = diff * color;
    // Specular    
    vec3 reflectDir = reflect(-lightDir, normal);
    vec3 halfwayDir = normalize(lightDir + viewDir);  
    float spec = pow(max(dot(normal, halfwayDir), 0.0), 32.0);

    vec3 specular = vec3(0.2) * spec;
    gl_FragColor = vec4(ambient + diffuse + specular, 1.0);
}
]]
}
```

I used the [CrazyBump](http://crazybump.com) to generate the `normal map`(img2) and `height map`(img3) from the `texture`(img1). 

Here is the images used in the code, you can download and use them directly:


img1:

![](https://github.com/FreeBlues/Codea-ParallaxMapping/blob/master/images/IMG1.JPG)

img2:

![](https://github.com/FreeBlues/Codea-ParallaxMapping/blob/master/images/IMG2.PNG)

img3:

![](https://github.com/FreeBlues/Codea-ParallaxMapping/blob/master/images/IMG3.PNG)


##	Reference

[38 视差贴图](http://bullteacher.com/38-parallax-mapping.html)  
[视差贴图（Parallax Mapping）与陡峭视差贴图(Steep Palallax Mapping)](http://blog.csdn.net/xiaoge132/article/details/51173002)  
[Parallax Occlusion Mapping in GLSL](http://sunandblackcat.com/tipFullView.php?topicid=28)   
[Jim's GameDev Blog: 置换贴图 Displacement Mapping](https://chengkehan.github.io/DisplacementMapping.html)
