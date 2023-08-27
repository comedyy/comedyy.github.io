1. unity使用Handles.FreeMoveHandle 来实现在scene视图上拖动物体

gizmos更多是用来绘制静态的对象。
而handles是用来绘制可交互的对象。 比如拖动，旋转，缩放等。

 OnDrawGizmos 跟 OnDrawGizmosSelected 都是gizmos的方法。OnDrawGizmosSelected 需要选中。
Gizmos.color .matrix 用来设置当前颜色跟变换举证。

```
DrawCube	Draw a solid box with center and size.
DrawFrustum	Draw a camera frustum using the currently set Gizmos.matrix for it's location and rotation.
DrawGUITexture	Draw a texture in the Scene.
DrawIcon	Draw an icon at a position in the Scene view.
DrawLine	Draws a line starting at from towards to.
DrawMesh	Draws a mesh.
DrawRay	Draws a ray starting at from to from + direction.
DrawSphere	Draws a solid sphere with center and radius.

DrawWireCube	Draw a wireframe box with center and size.
DrawWireMesh	Draws a wireframe mesh.
DrawWireSphere	Draws a wireframe sphere with center and radius.
```

Handles的api
Vector3 p = Handles.FreeMoveHandle(example.targetPosition, Quaternion.identity, size, snap, Handles.ArrowHandleCap); // 绘制一个3D可以移动的对象
Quaternion rot =  FreeRotateHandle(Quaternion rotation, Vector3 position, float size); // 旋转
bool x = Handles.Button() // 绘制一个3D按钮
Quaternion rot = Handles.Disc()

var pos = Handles.PositionHandle()
var radius = Handles.RadiusHandle()
var rot = Handles.RotationHandle()
var scale = Handles.ScaleHandle() 
var x = Handles.ScaleSlider()// 单一维度的大小
var x = Handles.ScaleValueHandle()// 变大变小之类的。
Handles.TransformHandle() 整个transform都可以


Handles.DrawBezier
Handles.DrawCamera 是要把camera的信息画在屏幕上吗
Handles.DrawDottedLine 画点线
Handles.DrawDottedLines
Handles.DrawLine
Handles.DrawLines
Handles.DrawPolyLine 
Handles.DrawAAPolyLine // 有抗锯齿的
Handles.DrawAAConvexPolygon // 有抗锯齿的多边形
Handles.DrawSolidArc // 弧形
Handles.DrawWireArc // wire
Handles.DrawSolidDisc // 圆盘
Handles.DrawWireDisc // wire
Handles.DrawSolidRectangleWithOutline // rect带outline
Handles.DrawWireCube
Handles.Label


Handles.DrawSelectionFrame ?
Handles.SnapToGrid() // 设置对其？



后面的Handles.ArrowHandleCap 还可以替换成 
Handles.SphereHandleCap
Handles.RectangleHandleCap
Handles.DotHandleCap
Handles.ConeHandleCap
Handles.CubeHandleCap
Handles.CylinderHandleCap
Handles.CircleHandleCap
Handles.ArrowHandleCap

Handles.BeginGUI 是可以让你在Scene面板上绘制一个button。
```
if (GUILayout.Button("Reset Area", GUILayout.Width(100)))
{
    handleExample.shieldArea = 5;
}
```

Handles.GetMainGameViewSize 





