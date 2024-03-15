---
title: CharacterController移动与跳跃示例代码
date: 2024-02-02
---

# CharacterController移动与跳跃示例代码

```c#
using UnityEngine;

public class MarchSevenController : MonoBehaviour
{
    private CharacterController playerController;
    private Vector3 playerVelocity;
    private bool groundedPlayer;
    private float playerSpeed = 2.0f;
    private float jumpHeight = 1.0f;
    private float gravityValue = -9.81f;
    private bool isStanding = true;

    // Start is called before the first frame update
    void Start()
    {
        playerController = GetComponent<CharacterController>();
    }

    // Update is called once per frame
    void Update()
    {
        // 使用 CharacterController 检查玩家是否在地面上
        groundedPlayer = playerController.isGrounded;

        // 如果玩家在地面上且垂直速度向下，重置 y 轴速度
        if (groundedPlayer && playerVelocity.y < 0)
        {
            playerVelocity.y = 0f;
        }

        // 获取水平输入
        Vector3 move = new Vector3(Input.GetAxis("Horizontal"), 0, Input.GetAxis("Vertical")); // 位置向量
        // unity使用的是左手坐标系
        // 前后是x，左右是z，上下是y

        //若有位移，则不站立
        if (move != Vector3.zero)
        {
            isStanding = false;
        }

        //若无位移，则站立
        if (move == Vector3.zero && playerVelocity.y == 0f)
        {
            isStanding = true;
        }

        // 检查是否有垂直输入（跳跃）及是否满足触发条件（在地上｜站立）
        if (Input.GetButtonDown("Jump") && (groundedPlayer || isStanding))
        {
            // 计算垂直的加速度
            playerVelocity.y += Mathf.Sqrt(2 * jumpHeight * -gravityValue);
            //不站立了
            isStanding = false;
        }

        // 每帧应用重力到垂直速度分量
        playerVelocity.y += gravityValue * Time.deltaTime;

        // 通过计算出的移动向量移动角色控制器
      	// Move用到了y=a(t-t0)的公式，这里playerSpeed与playerVelocity.y代表了a，Time.deltaTime代表了t-t0。
      	// 而move是一个(-1,1)的参数，控制方向和力度
        playerController.Move((move * playerSpeed + new Vector3(0, playerVelocity.y, 0)) * Time.deltaTime);
    }
}
```

**注意事项**

**1、垂直加速度用到了物理公式**

原始公式用于计算跳跃时所需的初速度，基于动能和势能的等价转换。这个公式是从基本的物理原理中得出的，特别是考虑到在跳跃的最高点，所有的动能（由于速度产生）都转化为势能（由于高度产生）。这个关系可以通过以下等式表示：

$$
\frac{1}{2} mv^2 = mgh
$$

变换后得到

$$
v = \sqrt{2gh}
$$

**2、关于isGrounded**

官方描述：Was the CharacterController touching the ground during the last move。

这意味着 `CharacterController` 的 `isGrounded` 状态是在最后一次执行 `Move` 方法后更新的。

这就有一个问题，如果在单个 `Update` 周期内执行了两次 `Move` 方法，或者角色保持静止而没有进行移动（即没有调用 `Move` 方法），则`isGrounded`的判断可能会出错。因此，`isGrounded`只能判断移动过程中的站立状态，并且还要求只有一次Move的调用。

在代码处理上：

1. **合并移动指令：** 将水平和垂直移动的逻辑合并，确保每个 `Update` 循环中只调用一次 `Move` 方法。
2. **增加站立状态判断：** 引入一个站立状态的变量判断角色是否可以跳跃。