# skinning 的原理。

1. 我们省去了 在cpu更新mesh的操作，而是在gpu中使用顶点着色器来处理。
    1. 输入数据： 原本mesh，包含顶点，uv，以及一个骨骼权重（boneIndex1, boneIndex2, weight）, 以及这个动画的所有骨骼的Matrix,写入图片。 还有就是当前的动画的进度。
    2. 我们骨骼是由一个数组的，boneIndex就是下标。我们每个顶点的骨骼权重原本是在mesh的skin上，我们就把它读取出来放入到我们自定义的一个结构中，比如normal中。
    3. 我们需要把动画bake到一张贴图中，我们要bake什么数据进去呢。是每一帧的每个骨骼的matrix。那这个matrix怎么计算呢。
        renderer.WorldToLocal * bone.transform.localToWorld * bonepose.matrix 就得到了 一个顶点的变换矩阵。 乘以顶点的位置.
        为什么可以直接乘以顶点位置。 bonepose.materix * 顶点位置 = 顶点在骨骼空间的位置。 （这个可以考虑通过打印出来，看一下是怎么变化的）。
        要再乘以骨骼到世界空间的坐标，差不多就是一个网格的正常形状。

        我们在cpu端模拟了一下 顶点的转换过程。
        ```
        for(int i = 0; i < vertex.Length; i++)
        {
            var vetex = vertex[i];
            var bone0 = boneWeight[i].boneIndex0;
            var bone1 = boneWeight[i].boneIndex1;
            var bone2 = boneWeight[i].boneIndex2;
            var bone3 = boneWeight[i].boneIndex3;
            var weight0 = boneWeight[i].weight0;
            var weight1 = boneWeight[i].weight1;
            var weight2 = boneWeight[i].weight2;
            var weight3 = boneWeight[i].weight3;

            var pos = (list[bone0].localToWorldMatrix *  mesh.bindposes[bone0]).MultiplyPoint(vetex) * weight0 
                    + (list[bone1].localToWorldMatrix *  mesh.bindposes[bone1]).MultiplyPoint(vetex) * weight1 
                    + (list[bone2].localToWorldMatrix *  mesh.bindposes[bone2]).MultiplyPoint(vetex) * weight2 
                    + (list[bone3].localToWorldMatrix *  mesh.bindposes[bone3]).MultiplyPoint(vetex) * weight3;

            Gizmos.DrawSphere(pos, 0.05f);
        }
        ```
        这样就可以基本绘制出一个mesh了。


    4. 我们在shader中要如何处理。
        我们把（boneIndex0, boneIndex1, weight）通过Normal属性带入到mesh中，然后在顶点着色器上，我们处理：
        从动画贴图中，读取出当前vertex对应的bone的matrix，然后计算出顶点的模型坐标。

    5. 我们可以从中学到啥：
        1. Mesh中包含一个 BoneWeight, 专门给骨骼使用，还有一个bindposes，这两个跟蒙皮动画关系很紧密。
        2. unity中的fbx中可以设置动画受到多少根骨骼的影响。我们要确保所有的动画骨骼数量都一样。
        3. 我们可以定制mesh里面放哪些东西。把你需要的数据放进去，然后再shader中，通过appdata 使用。