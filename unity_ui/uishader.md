## ui shader
在构建uishader的时候，有几个值需要注意一下。

1. _TextureSampleAdd 如果是使用Alhpa8，这个值是(1, 1, 1, 0), 否者是 （0, 0, 0, 0）
2. _MainTex_TexelSize 假设图片占据屏幕的大小是 (width, height) 像素，那么这个值是 (1/width, 1/height, width, height),这个值可以用来做高斯模糊，更快定位出周围的像素。 如果一张图片占据屏幕width长度，而一张图片的x轴方向对uv最大值是1，那么每 1/width的uv距离一个像素。
``` 
// 方向模糊。
v2f vert(appdata_t v)
{
    v2f o;
    o.pos = UnityObjectToClipPos(v.vertex);
    o.uv = v.texcoord.xy;
    o.color = v.color * _Color;

    half2 offs = _MainTex_TexelSize.xy*half2(_DirectionX, _DirectionY)*_BlurSize;
    o.uvoff[0] = v.texcoord.xyxy + offs.xyxy*coordOffset * 3;
    o.uvoff[1] = v.texcoord.xyxy + offs.xyxy*coordOffset * 2;
    o.uvoff[2] = v.texcoord.xyxy + offs.xyxy*coordOffset;
    o.worldPosition = v.vertex;

    return o;
}

fixed4 frag(v2f i) :SV_Target
{
    fixed4 c = (tex2D(_MainTex,i.uv)+ _TextureSampleAdd)* i.color*weight[3];
    
    for (int idx = 0; idx < 3; idx++)
    {
        c += (tex2D(_MainTex,i.uvoff[idx].xy)+ _TextureSampleAdd)* i.color*weight[idx];
        c += (tex2D(_MainTex,i.uvoff[idx].zw)+ _TextureSampleAdd)* i.color*weight[idx];
    }
			
```