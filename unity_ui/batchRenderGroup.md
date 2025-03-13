# batch render group

这个似乎结合了 drawMeshInstanced 跟drawMeshInstanceIndirect的有点。
    1. drawMeshInstanced 优点是兼容性好，但是性能不够好，draw的时候带着数据一起draw。
    2. drawMeshInstanceIndirect 效率是好，但是有些opengl3.0的顶点着色器访问不了ssbo。所以兼容性有问题。
    3. batchRenderGroup 它在opengl3.0的api下使用ubo来代替ssbo，但是ubo的大小有限，需要使用多个batch来提交，它跟drawMeshInstancedIndirect 一样，会使用graphicBuffer来把玩家数据先提交到gpu，然后再使用api对这些数据进行绘制。
        1. 如果这些数据没有变动，就无需重新upload。（但是似乎经常都是由变动。）
        2. 可以选择性的进行绘制，可以指定visibleInstances， 只渲染visibleInstances对象。
        3. 更重要的是，你不需要对shader进行修改。如果你使用gpuinstance的话，你要对shader进行大规模修改，加入instance的macro。这比较麻烦。而brg可以直接使用simplelit的shader， 然后通过MetadataValue把属性写入到对应的shader属性中。
        
1. m_BatchRendererGroup = new BatchRendererGroup(this.OnPerformCulling, IntPtr.Zero); // 创建一个对象，再OnPerformCulling中进行回调绘制。
2. 通过判断是否是使用constbuffer来判断一次batch的窗口大小。
    UseConstantBuffer => BatchRendererGroup.BufferTarget == BatchBufferTarget.ConstantBuffer
    ```
    if (UseConstantBuffer)
    {
        m_alignedGPUWindowSize = BatchRendererGroup.GetConstantBufferMaxWindowSize();
        m_maxInstancePerWindow = m_alignedGPUWindowSize / instanceSize;
        m_windowCount = (m_maxInstances + m_maxInstancePerWindow - 1) / m_maxInstancePerWindow;
        m_totalGpuBufferSize = m_windowCount * m_alignedGPUWindowSize;
        m_GPUPersistentInstanceData = new GraphicsBuffer(GraphicsBuffer.Target.Constant, m_totalGpuBufferSize / 16, 16);
    }
    else
    {
        m_alignedGPUWindowSize = (m_maxInstances * instanceSize + 15) & (-16);
        m_maxInstancePerWindow = maxInstances;
        m_windowCount = 1;
        m_totalGpuBufferSize = m_windowCount * m_alignedGPUWindowSize;
        m_GPUPersistentInstanceData = new GraphicsBuffer(GraphicsBuffer.Target.Raw, m_totalGpuBufferSize / 4, 4);
    }

    ```
3. 创建batch。（告诉shader如何填充这些数据）
    int objectToWorldID = Shader.PropertyToID("unity_ObjectToWorld");
    int worldToObjectID = Shader.PropertyToID("unity_WorldToObject");
    int colorID = Shader.PropertyToID("_BaseColor");
    var batchMetadata = new NativeArray<MetadataValue>(3, Allocator.Temp, NativeArrayOptions.UninitializedMemory);
    batchMetadata[0] = CreateMetadataValue(objectToWorldID, 0, true);       // matrices
    batchMetadata[1] = CreateMetadataValue(worldToObjectID, m_maxInstancePerWindow * 3 * 16, true); // inverse matrices
    batchMetadata[2] = CreateMetadataValue(colorID, m_maxInstancePerWindow * 3 * 2 * 16, true); // colors
    m_batchIDs[b] = m_BatchRendererGroup.AddBatch(batchMetadata, m_GPUPersistentInstanceData.bufferHandle, (uint)offset, UseConstantBuffer ? (uint)m_alignedGPUWindowSize : 0);
4. m_BatchRendererGroup.SetGlobalBounds(bounds); // 设置超大的范围，不会被裁剪。

5. 把逻辑位置写入进入
```
 float3x3 rot = item.mat;
    // compute the new current frame matrix
    _sysmemBuffer[(windowOffsetInFloat4 + i * 3 + 0)] = new float4(rot.c0.x, rot.c0.y, rot.c0.z, rot.c1.x);
    _sysmemBuffer[(windowOffsetInFloat4 + i * 3 + 1)] = new float4(rot.c1.y, rot.c1.z, rot.c2.x, rot.c2.y);
    _sysmemBuffer[(windowOffsetInFloat4 + i * 3 + 2)] = new float4(rot.c2.z, bpos.x, bpos.y, bpos.z);

    // compute the new inverse matrix
    _sysmemBuffer[(windowOffsetInFloat4 + _maxInstancePerWindow * 3 * 1 + i * 3 + 0)] = new float4(rot.c0.x, rot.c1.x, rot.c2.x, rot.c0.y);
    _sysmemBuffer[(windowOffsetInFloat4 + _maxInstancePerWindow * 3 * 1 + i * 3 + 1)] = new float4(rot.c1.y, rot.c2.y, rot.c0.z, rot.c1.z);
    _sysmemBuffer[(windowOffsetInFloat4 + _maxInstancePerWindow * 3 * 1 + i * 3 + 2)] = new float4(rot.c2.z, -bpos.x, -bpos.y, -bpos.z);

    // update colors
    _sysmemBuffer[windowOffsetInFloat4 + _maxInstancePerWindow * 3 * 2 + i] = new float4(item.color, 1);
```
