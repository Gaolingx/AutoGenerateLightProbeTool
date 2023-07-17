# AutoGenerateLightProbeTool
 This is a tool for generate light probe in unity automatically.

## 1. 概念

首先，我们来回顾全局光照（Global Illumination ）的设置，就是为了获取反射光影效果，增强场景真实性

| 光照模式 | 全局光照设置 （Global Illumination） | 物体设置   | 间接（反射）光影效果 | 是否产生烘焙贴图           | 直接光影效果 |
| -------- | ------------------------------------ | ---------- | -------------------- | -------------------------- | ------------ |
| realtime | Realtime Global Illumination         | not static | 有效果               | 无烘焙贴图，但会有实时贴图 | 有           |
| realtime | Mixed Global Illumination            | static     | 无效果               | 有烘焙贴图                 | 有           |
| baked    | Mixed Global Illumination            | not static | 无效果               | 无烘焙贴图                 | 无           |
| baked    | Mixed Global Illumination            | static     | 有效果               | 有烘焙贴图                 | 有           |
| Mixed    | Mixed Global Illumination            | not static | 无效果               | 无烘焙贴图                 | 有           |
| Mixed    | Mixed Global Illumination            | static     | 有效果               | 有烘焙贴图                 | 有           |

图1：静态、非静态物体，都没有配置好全局光照（间接光照）的效果

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/全无.png)

图2：静态物体有全局光照（间接光照）、非静态物体无间接光照的效果

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/静态有动态无.png)

图3：使用光照探针配合光照贴图，静态、非静态物体，都有间接光照效果

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/全有.png)

> 注意：
> * 实时光照，如果想要有间接光影效果，必须将全局光照配置为 Realtime Global Illumination
> * 实时光照，如果想要有间接光影效果，还需要将物体设置为 Contribute Global Illumination 
    ![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/cbi.png)
> * 但实际游戏中，是不会使用实时全局光照  Realtime Global Illumination(耗费大，而且效果差并非完全实时)，所以要用 Mixed 光照配合 Mixed Global Illumination
> * 为了弥补 非静态物体的间接光照效果，需要使用光照探针

间接光影效果最佳方案总结：
1. 场景光照设置为 Mixed Global Illumination ；
2. 光源一般如果同时覆盖动态和静态物体，一定要设置为 Mixed 模式；
3. 物体和光源都设置好后，在场景中添加光照探针组，并且 Generate LightMap 生成光照贴图
4. 享受最合理的间接光影效果~

### 1.1 什么是光照探针

* 实时 Realtime 光照太浪费资源，一般不会大范围使用在实际游戏场景中；
* 烘焙 Baked 光照又不能作用在动态物体上，这就有了光照探针，来弥补光照贴图的不足
* 对于 Mixed 光照，虽然可以将光影效果投射到移动物体上，但移动物体不会从静态环境中接收到反射光，所以也需要使用光照探针

未添加 光照探针：

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/beforeLP.png)

正在添加 光照探针：

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/AddingLP.png)

添加光照探针后：

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/全有.png)

光照探针大致原理：在某一光照探针的所在位置点上对光照信息进行采样，然后从该光照探针相邻的其他光照探针的位置上对光照信息进行采样，把这些采样得到的光照信息进行插值运算，便可算出这些光照探针之间某个位置的光照信息。

### 1.2 光照探针作用

光探测器（光照探针 Light Probe）存储有关场景中照明的“烘焙”信息。

光照贴图存储有关光线照射场景中表面的光照信息，但光照探测器存储有关光线穿过场景中空白空间的信息。

说白了，光照贴图只存储 mesh 表面的光影信息；而光照探针，存储空白空间的光影信息，包括直接光和间接光（反射光），是对光照贴图的补充

## 2. 使用 光照探针组  Light Probe Group

### 2.1 概念

光照探针组就是一组光照探针，默认是一个立方体，4*2一组，共八个光照探针组成。

可以通过编辑，增加或删除光照探针，也可以随意移动光照探针位置

### 2.2 使用方式

可以从菜单中添加 Light Probe Group 到当前场景中：Component > Rendering > Light Probe Group

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/addLPG.png)

光照探针组初始形态：

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/LPG.png)

### 2.3 属性

光照探针组组件在 Inspector 窗口如下：

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/class-LightProbeGroup-20183.png)

* Edit Light Probes: 编辑光照探针，点击后，可以在场景中编辑光照探针组中的光照探针，进行增加、删除，移动位置等操作
* Show Wireframe ：显示光照探针之间的连线组成的线框，如果禁用，则只显示探针所在点，而不显示之间的连线；
* Remove Ringing ：去除振铃。启用此属性后，Unity 会自动从场景中移除 Light Probe 振铃。  
  在某些情况下，Light Probe 会表现出一种称为“振铃”的不良行为。当 Light Probe 周围的光线存在显着差异时，通常会发生这种情况。例如，如果光探头的一侧有亮光，而另一侧没有光，则光强度可能会在背面“过冲”。这种过冲会在背面产生一个光点。
    ![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/class-LightProbeGroup-Ringing.png)  
    启用Remove Ringing。Unity 会自动移除不希望出现的光点。但是，这通常会降低 Light Probes 的准确度，并降低光线对比度，因此您必须检查视觉结果
* Selected Probe Position : 选定光照探针所在位置

### 2.4 光探头放置故障排除

对 Light Probe 位置的选择必须考虑到光照是在 Light Probe 组之间插值的。如果您的 Light Probe 没有充分覆盖整个场景中的光照变化，则可能会出现问题。

下面的示例显示了一个夜间场景，两侧有两个明亮的路灯，中间有一个黑暗区域。如果 Light Probe 仅放置在路灯附近，而在黑暗区域没有放置，则来自灯的照明直接穿过黑暗的缝隙，照射到移动的物体上。这是因为照明从一个亮点插入到另一个亮点，中间没有关于暗区的信息。

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/class-LightProbeGroup-12.png)

如果您使用的是实时或混合光，则此问题可能不太明显，因为只有间接光会穿过间隙。如果您完全使用烘焙光，问题会更加明显
，因为在这种情况下，移动物体上的直射光也是由 Light Probes 插入的。在此示例场景中，两个灯被烘焙，因此移动对象从 Light Probes 获得直接光。在这里您可以看到结果 - 移动的物体（救护车）在穿过黑暗区域时保持明亮，这不是预期的效果。黄色线框四面体显示插值发生在街道明亮的一端与另一端之间。

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/class-LightProbeGroup-13.png)

这是一个不希望的效果 - 救护车在通过黑暗区域时仍然保持明亮，因为没有在黑暗区域放置光探头。

为了解决这个问题，你应该在黑暗区域放置更多的Light Probes，如下图：

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/class-LightProbeGroup-14.png)

现在场景在黑暗区域也有光探针。因此，移动的救护车在从场景的一侧行驶到另一侧时会呈现较暗的灯光。

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/class-LightProbeGroup-15.png)

## 3. 用代码添加光照探针

在比较大的场景中，如果需要光照探针时，还是使用手动添加，不仅位置不会很精确，而且会非常耗时。

这种情况下，就可以使用代码来自动防止光照探针：
1. 添加 LightProbeGroup 游戏对象；
2. 在代码中操作 LightProbeGroup 游戏对象，为其中用代码精准添加并布置光照探针

下面是核心功能代码：
``` C#
 //获取光照探针组件
  LightProbeGroup lightProbeGroup = GetComponent<LightProbeGroup>();
  //建立要插入探针位置的列表
  List<Vector3> positions = new List<Vector3>();
  // 下面就是向 positions 中添加合理的位置点
  .......
  .......
  //向光照探针组中，添加探针
  lightProbeGroup.probePositions = positions.ToArray(); 
```

完整实例：下面脚本可以将 Light Probes 放置在一个圆圈或一个环中

![](https://raw.githubusercontent.com/Gaolingx/AutoGenerateLightProbeTool/main/imgs/glpg.png)

``` C#
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(LightProbeGroup))]
public class LightProbesTetrahedralGrid : MonoBehaviour
{
    // Common
    [Tooltip("边长 : 同层探针间连线长度")]
    public float m_Side = 1.0f;
    [Tooltip("半径 : 圆形探针组大圆半径")]
    public float m_Radius = 5.0f;
    [Tooltip("内半径 : 圆形探针组内部小圆半径")]
    public float m_InnerRadius = 0.1f;
    [Tooltip("高度 : 圆形探针组高度")]
    public float m_Height = 2.0f;
    [Tooltip("层数 : 圆形探针组层数")]
    public uint m_Levels = 3;
    //最小半径
    const float kMinSide = 0.05f;
    //最低高度
    const float kMinHeight = 0.05f;
    //最小内圆半径
    const float kMinInnerRadius = 0.1f;
    //最小迭代数
    const uint kMinIterations = 4;

    // 游戏运行时，先调用 Generate 方法，添加光照探针到场景中
    private void FixedUpdate()
    {
        Generate();
    }

    // OnValidate可以用来验证一些数据，脚本加载或Inspector中的任何值被修改时会调用
    // 可以作为数据的保护，避免一些字段（或属性）被设置为不可用（不合理）的数据值
    public void OnValidate()
    {
        //保证 m_Side 不小于 kMinSide
        m_Side = Mathf.Max(kMinSide, m_Side);
        //保证 m_Height 不小于 kMinHeight
        m_Height = Mathf.Max(kMinHeight, m_Height);
        // 对  m_Radius 和 m_InnerRadius 进行限制
        if (m_InnerRadius < kMinInnerRadius)
        {
            TriangleProps props = new TriangleProps(m_Side);
            m_Radius = Mathf.Max(props.circumscribedCircleRadius + 0.01f, m_Radius);
        }
        else
        {
            m_Radius = Mathf.Max(0.1f, m_Radius);
            m_InnerRadius = Mathf.Min(m_Radius, m_InnerRadius);
        }
    }

    // 结构体 三角形探针
    struct TriangleProps
    {
        public TriangleProps(float triangleSide)
        {
            side = triangleSide;
            halfSide = side / 2.0f;
            height = Mathf.Sqrt(3.0f) * side / 2.0f;
            //内切圆半径
            inscribedCircleRadius = Mathf.Sqrt(3.0f) * side / 6.0f;
            //外切圆半径
            circumscribedCircleRadius = 2.0f * height / 3.0f;
        }
        public float side;
        public float halfSide;
        public float height;
        public float inscribedCircleRadius;
        public float circumscribedCircleRadius;
    };
    //三角形探针位，用来辅助生成探针的位置
    private TriangleProps m_TriangleProps;
    //生成探针主方法
    public void Generate()
    {
        //获取光照探针组件
        LightProbeGroup lightProbeGroup = GetComponent<LightProbeGroup>();
        //建立要插入探针位置的列表
        List<Vector3> positions = new List<Vector3>();
        //三角形探针位，用来辅助生成探针的位置
        m_TriangleProps = new TriangleProps(m_Side);

        if (m_InnerRadius < kMinInnerRadius)
            //生成圆柱
            GenerateCylinder(m_TriangleProps, m_Radius, m_Height, m_Levels, positions);
        else
            //生成环
            GenerateRing(m_TriangleProps, m_Radius, m_InnerRadius, m_Height, m_Levels, positions);
        //向光照探针组中，添加探针
        lightProbeGroup.probePositions = positions.ToArray();
    }
    //尝试添加
    static void AttemptAdding(Vector3 position, Vector3 center, float distanceCutoffSquared, List<Vector3> outPositions)
    {
        if ((position - center).sqrMagnitude < distanceCutoffSquared)
            outPositions.Add(position);
    }
    //计算圆柱体迭代
    uint CalculateCylinderIterations(TriangleProps props, float radius)
    {
        int iterations = Mathf.CeilToInt((radius + props.height - props.inscribedCircleRadius) / props.height);
        if (iterations > 0)
            return (uint)iterations;
        return 0;
    }
    //生成圆柱体
    void GenerateCylinder(TriangleProps props, float radius, float height, uint levels, List<Vector3> outPositions)
    {
        uint iterations = CalculateCylinderIterations(props, radius);
        float distanceCutoff = radius;
        float distanceCutoffSquared = distanceCutoff * distanceCutoff;
        Vector3 up = new Vector3(props.circumscribedCircleRadius, 0.0f, 0.0f);
        Vector3 leftDown = new Vector3(-props.inscribedCircleRadius, 0.0f, -props.halfSide);
        Vector3 rightDown = new Vector3(-props.inscribedCircleRadius, 0.0f, props.halfSide);
        for (uint l = 0; l < levels; l++)
        {
            float tLevel = levels == 1 ? 0 : (float)l / (float)(levels - 1);
            Vector3 center = new Vector3(0.0f, tLevel * height, 0.0f);
            if (l % 2 == 0)
            {
                for (uint i = 0; i < iterations; i++)
                {
                    Vector3 upCorner = center + up + (float)i * up * 2.0f * 3.0f / 2.0f;
                    Vector3 leftDownCorner = center + leftDown + (float)i * leftDown * 2.0f * 3.0f / 2.0f;
                    Vector3 rightDownCorner = center + rightDown + (float)i * rightDown * 2.0f * 3.0f / 2.0f;
                    AttemptAdding(upCorner, center, distanceCutoffSquared, outPositions);
                    AttemptAdding(leftDownCorner, center, distanceCutoffSquared, outPositions);
                    AttemptAdding(rightDownCorner, center, distanceCutoffSquared, outPositions);
                    Vector3 leftDownUp = upCorner - leftDownCorner;
                    Vector3 upRightDown = rightDownCorner - upCorner;
                    Vector3 rightDownLeftDown = leftDownCorner - rightDownCorner;
                    uint subdiv = 3 * i + 1;
                    for (uint s = 1; s < subdiv; s++)
                    {
                        Vector3 leftDownUpSubdiv = leftDownCorner + leftDownUp * (float)s / (float)subdiv;
                        AttemptAdding(leftDownUpSubdiv, center, distanceCutoffSquared, outPositions);
                        Vector3 upRightDownSubdiv = upCorner + upRightDown * (float)s / (float)subdiv;
                        AttemptAdding(upRightDownSubdiv, center, distanceCutoffSquared, outPositions);
                        Vector3 rightDownLeftDownSubdiv = rightDownCorner + rightDownLeftDown * (float)s / (float)subdiv;
                        AttemptAdding(rightDownLeftDownSubdiv, center, distanceCutoffSquared, outPositions);
                    }
                }
            }
            else
            {
                for (uint i = 0; i < iterations; i++)
                {
                    Vector3 upCorner = center + (float)i * (2.0f * up * 3.0f / 2.0f);
                    Vector3 leftDownCorner = center + (float)i * (2.0f * leftDown * 3.0f / 2.0f);
                    Vector3 rightDownCorner = center + (float)i * (2.0f * rightDown * 3.0f / 2.0f);
                    AttemptAdding(upCorner, center, distanceCutoffSquared, outPositions);
                    AttemptAdding(leftDownCorner, center, distanceCutoffSquared, outPositions);
                    AttemptAdding(rightDownCorner, center, distanceCutoffSquared, outPositions);
                    Vector3 leftDownUp = upCorner - leftDownCorner;
                    Vector3 upRightDown = rightDownCorner - upCorner;
                    Vector3 rightDownLeftDown = leftDownCorner - rightDownCorner;
                    uint subdiv = 3 * i;
                    for (uint s = 1; s < subdiv; s++)
                    {
                        Vector3 leftDownUpSubdiv = leftDownCorner + leftDownUp * (float)s / (float)subdiv;
                        AttemptAdding(leftDownUpSubdiv, center, distanceCutoffSquared, outPositions);
                        Vector3 upRightDownSubdiv = upCorner + upRightDown * (float)s / (float)subdiv;
                        AttemptAdding(upRightDownSubdiv, center, distanceCutoffSquared, outPositions);
                        Vector3 rightDownLeftDownSubdiv = rightDownCorner + rightDownLeftDown * (float)s / (float)subdiv;
                        AttemptAdding(rightDownLeftDownSubdiv, center, distanceCutoffSquared, outPositions);
                    }
                }
            }
        }
    }
    //生成环
    void GenerateRing(TriangleProps props, float radius, float innerRadius, float height, uint levels, List<Vector3> outPositions)
    {
        float chordLength = props.side;
        float angle = Mathf.Clamp(2.0f * Mathf.Asin(chordLength / (2.0f * radius)), 0.01f, 2.0f * Mathf.PI);
        uint slicesAtRadius = (uint)Mathf.FloorToInt(2.0f * Mathf.PI / angle);
        uint layers = (uint)Mathf.Max(Mathf.Ceil((radius - innerRadius) / props.height), 0.0f);
        for (uint level = 0; level < levels; level++)
        {
            float tLevel = levels == 1 ? 0 : (float)level / (float)(levels - 1);
            float y = height * tLevel;
            float iterationOffset0 = level % 2 == 0 ? 0.0f : 0.5f;
            for (uint layer = 0; layer < layers; layer++)
            {
                float tLayer = layers == 1 ? 1.0f : (float)layer / (float)(layers - 1);
                float tIterations = (tLayer * (radius - innerRadius) + innerRadius - kMinInnerRadius) / (radius - kMinInnerRadius);
                uint slices = (uint)Mathf.CeilToInt(Mathf.Lerp(kMinIterations, slicesAtRadius, tIterations));
                float x = innerRadius + (radius - innerRadius) * tLayer;
                Vector3 position = new Vector3(x, y, 0.0f);
                float layerSliceOffset = layer % 2 == 0 ? 0.0f : 0.5f;
                for (uint slice = 0; slice < slices; slice++)
                {
                    Quaternion rotation = Quaternion.Euler(0.0f, (slice + iterationOffset0 + layerSliceOffset) * 360.0f / (float)slices, 0.0f);
                    outPositions.Add(rotation * position);
                }
            }
        }
    }

}
```

<br>
<hr>
<br>

> 参考资料：
> * [光照探针文档 - Unity 官方](https://docs.unity3d.com/2022.2/Documentation/Manual/LightProbes.html)
> * [Brackeys - youtube - HIGH QUALITY LIGHTING using Light Probes](https://www.youtube.com/watch?v=_E0JXOZDTKA)

<br>
<hr>
<br>