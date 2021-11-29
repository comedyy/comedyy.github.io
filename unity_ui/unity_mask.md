### unity mask

unity中的mask或者rectmask2d都是用于裁剪



#### mask

1. 它使用alphaclip，把自己所在的区域alpha很小的区域剔除掉。
2. 它使用模板测试，把自己所在的区域没有被剔除掉的位置模板值标记为1
3. 它的子节点使用模板测试，只有在模板值==1的时候，才通过测试，显示像素点。
4. 通过canvasrenderer来实现退出当前对象的渲染的时候，其他节点更换为普通材质。
5. 通过colormask来实现 是否显示 mask对应图片的功能。



#### rectmask

1. 查找所有的子对象，给他们的canvasrenderer添加一个 裁剪区域。
2. 这个裁剪区域在shader中计算。如果当前像素在裁剪区域内，那么alpha = 0；



