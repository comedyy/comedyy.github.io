# gpu instancing

1.  使用Graphics.DrawMeshInstancedIndirect()来绘制。通常使用ssbo来进行传递参数。
    使用mesh，material进行渲染。
    需要支持computeshader。 使用systeminfo.supportComputeShader来获取。

我看到shader中使用ssbo的时候要求target > 4.5

#pragma target 4.5（或 es3.1）
OpenGL ES 3.1 功能（D3D 平台上的 DX11 SM5.0，只是没有曲面细分着色器）。
在早于 SM5.0 的 DX11、早于 4.3 的 OpenGL（即 Mac）和 OpenGL ES 2.0/3.0 上不支持。
在 DX11+ SM5.0、OpenGL 4.3+、OpenGL ES 3.1、Metal、Vulkan 和 PS4/XB1 游戏主机上支持。
有计算着色器、随机访问纹理写入、原子等。没有几何着色器和曲面细分着色器。

所以就是 opengl3.1以上就可以使用。


1. 准备数据。
    需要准备gpuinstance的数据，主要是位置，朝向，以及动画帧之类的数据。
    这些数据对应于shader是： StructuredBuffer<float3> objectPositionBuffer;
    程序往material里面初始化buffer：m_Material.SetBuffer("objectPositionBuffer", m_ObjectPositionBuffer);
    然后每帧修改buffer的数据： m_ObjectPositionBuffer.SetData(array, 0, 0, count);
    然后shader里面就可以通过gpuinstancing的函数获取奥position； float3 objectPosition = objectPositionBuffer[unity_InstanceID];

2. shader中的配置注意：如果使用Graphics.DrawMeshInstancedIndirect 来绘制。
    1. #pragma instancing_options procedural:setup，定义一个setup函数，它用来每个实力的调用。（目前看来主要在surface shader中使用）
    2. 调用了setup之后，它就会帮你把 UNITY_PROCEDURAL_INSTANCING_ENABLED 宏弄好
    3. 使用UNITY_SETUP_INSTANCE_ID(v); 初始化 unity_InstanceID值，以后你就可以使用了。
    4. UNITY_SETUP_INSTANCE_ID(v) 需要在 UNITY_PROCEDURAL_INSTANCING_ENABLED 中调用，才能使用正确的宏分支，才能正确定义 unity_InstanceID
    5. 注意要在appdata中，加上 UNITY_VERTEX_INPUT_INSTANCE_ID， 这样调用 UNITY_SETUP_INSTANCE_ID()才能正确的设置 unity_InstanceID
    6. 如果你想在fragmentshader中使用unity_IntanceId的话，需要调用 UNITY_TRANSFER_INSTANCE_ID(input, output); 这样在fragmentshader中使用 UNITY_SETUP_INSTANCE_ID(input); 初始化 unity_InstanceID。

3. 调用。
    每帧 设置compute buffer的数据。
    比如 positionbuffer，rotation，之类的数据。 同步设置会触发上传gpu的消耗，它有异步的接口。
    最后使用 Graphics.DrawMeshInstancedIndirect(m_Mesh, 0, m_Material, m_Bounds, m_ArgsBuffer, 0, m_MaterialPropertyBlock, UnityEngine.Rendering.ShadowCastingMode.Off, true, 0, m_RenderTargetCamera); 来调用函数。
    m_ArgsBuffer 也是一个computebuffer，用来存放一些调用参数。比如（mesh的index数量，调用的instance数量，0， 0， 0）

4. 总结，以及思考整个流程。
    1. 为啥会是这个流程。 用ssbo这个是很好理解的，我们只要把mesh，material，以及对应的ssbo上传到gpu中，gpu通过我们的数据进行绘制就可以了。
        unity往我们的appData中，塞入了一个instanceId，这个东西，这样我们 UNITY_SETUP_INSTANCE_ID 之后就可以使用它取值到正确的 位置信息。但是为啥还要使用转化一次呢，我们不能直接使用吗？  
        我看了一下实现 UNITY_SETUP_INSTANCE_ID 
        1. 通过appdata中的instanceid，计算出unity_InstanceID.
        2. 会去调用 UNITY_INSTANCING_PROCEDURAL_FUNC，这个就是我们 #pragma instancing_options procedural:setup 对应的setup。
        3. 同时它还会调用UnitySetupCompoundMatrices，具体的逻辑就是帮你设置好 那些关键矩阵，主要是跟model相关的。（新版本没有这个消耗了）

    2.如果想在fragmentshader中使用，需要fragment输入也包含UNITY_VERTEX_INPUT_INSTANCE_ID，然后使用UNITY_TRANSFER_INSTANCE_ID() 从appdata传递给fragment的输入。再在fragmentshader中使用UNITY_SETUP_INSTANCE_ID() 来初始化unity_InstanceID。
