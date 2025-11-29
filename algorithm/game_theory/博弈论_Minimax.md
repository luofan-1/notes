# Minimax 算法

> 2025/11/29 clcwdans

**题目：预测赢家 **

- 【题目描述】
  > 给你一个整数数组 ```nums```。玩家 ```1``` 和玩家 ```2``` 基于这个数组设计了一个游戏。玩家 ```1``` 和玩家 ```2``` 轮流进行自己的回合，玩家 ```1``` 先手。开始时，两个玩家的初始分值都是 ```0``` 。每一回合，玩家从数组的任意一端取一个数字（即 ```nums[0]``` 或 ```nums[nums.length - 1]```），取到的数字将会从数组中移除（数组长度减 ```1``` ）。玩家选中的数字将会加到他的得分上。当数组中没有剩余数字可取时，游戏结束。如果玩家 ```1``` 能成为赢家，返回 ```true``` 。如果两个玩家得分相等，同样认为玩家 ```1``` 是游戏的赢家，也返回 ```true``` 。你可以假设每个玩家的玩法都会使他的分数最大化。 请你编写递归函数解决这个问题。
- 【输入描述】
  > 第一行为整数 ```n``` ，表示数组长度，
  > 第二行为数组 ```nums``` 的元素。
- 【输出描述】
  > ```true``` 或者 ```false```
```c
#include <stdio.h>

int find_max_difference(int *nums, int left, int right);
int predict_winner(int *nums, int left, int right);

int main() {
    int n;
    scanf("%d", &n);
    int nums[n];
    for (int i=0; i<n; i++) {
        scanf("%d", &nums[i]);
    }
    if (predict_winner(nums, 0, n-1)) {
        printf("true");
    } else {
        printf("false");
    }
    return 0;
}

int predict_winner(int *nums, int left, int right) {
    // 分差才是决策的关键
    int difference = find_max_difference(nums, left, right);
    return difference >= 0;
}

int find_max_difference(int *nums, int left, int right) {
    if (left == right) {
        return nums[left];
    }
    int left_differnce = nums[left] - find_max_difference(nums, left+1, right);
    int right_difference = nums[right] - find_max_difference(nums, left, right-1);
    return left_differnce > right_difference ? left_differnce:right_difference;
}
```

## Minimax (极小极大) 简介

Minimax（极小极大）算法是 **双人零和博弈（一方收益=另一方损失）** 的核心决策算法，核心逻辑是“最大化自己的最小收益，同时最小化对手的最大收益”——适用于双方轮流行动、均采用最优策略的场景（如棋类、取数游戏、博弈论问题），也是上一题“预测游戏赢家”的底层算法思想。 

###  一、核心本质：双人博弈的“换位思考”

- **零和博弈**：玩家1的收益 = 玩家2的损失（比如取数游戏中，玩家1多拿1分，玩家2就少1分）； 

- **轮流决策**：玩家1（最大化者，Max）和玩家2（最小化者，Min）交替行动；

- **最优策略**：双方都知道所有可能的结果，会选择对自己最有利的行动。   

算法的核心是：**当前玩家（Max/Min）会假设对手后续会采取最优策略，从而反向推导自己当前的最优选择**。 
###  二、关键概念：最大化者（Max）vs 最小化者（Min）
- **最大化者（Max）**：目标是“最大化最终收益”（如上一题的玩家1），在决策时会选择所有可能结果中的**最大值**；
- **最小化者（Min）**：目标是“最小化Max的最终收益”（如上一题的玩家2），在决策时会选择所有可能结果中的**最小值**。 
### 三、算法流程（以“取数游戏”为例） 
  假设数组 ```nums = [1,5,2]```，玩家1（Max）先手，玩家2（Min）后手，流程如下：
  1. **构建博弈树**：每个节点代表“当前剩余区间”，分支代表“当前玩家的选择”（取左/取右），叶子节点代表“游戏结束时的最终收益（Max的得分差）”； 

  2. **递归遍历博弈树**：   

       - Max节点（玩家1的回合）：选择子节点中的**最大值**（最大化自己的收益）；   

       - Min节点（玩家2的回合）：选择子节点中的**最小值**（最小化Max的收益）； 

  3. **回溯得到最优解**：从叶子节点反向推导，得到根节点（初始状态）的最优收益。 

     #### 博弈树示意（nums = [1,5,2]） 
     
     ``` 
     根节点（Max，[0,2]）→ 最终收益：-2（Min的选择） 
     ├─ 选择左1 → 进入Min节点（[1,2]）→ 子节点最小值：3 
     │  ├─ 选择左5 → 叶子节点：5-2=3（Max的得分差） 
     │  └─ 选择右2 → 叶子节点：2-5=-3 
     └─ 选择右2 → 进入Min节点（[0,1]）→ 子节点最小值：4   
        ├─ 选择左1 → 叶子节点：1-5=-4   
        └─ 选择右5 → 叶子节点：5-1=4 
     ```
     - 根节点（Max）的两个选项收益：`1 - 3 = -2`（选左1）、`2 - 4 = -2`（选右2），最终Max只能选收益-2（<0），所以玩家1输。 

### 四、通用算法框架（递归实现）
以“最大化者收益”为目标，递归函数返回当前玩家在当前状态下的**最优收益**： 
```python 
def minimax(state, is_max_player):
    # 1. 终止条件：当前状态是终局（如数组为空、棋类分出胜负）
    if is_terminal(state):
        return evaluate(state)  # 返回终局对Max的收益值（正=Max赢，负=Min赢）
    
    # 2. Max玩家回合：选择所有子状态的最大值
    if is_max_player:
        max_score = -float('inf')
        # 遍历所有可能的行动（如取左/取右、下棋的所有位置）
        for action in get_all_actions(state):
            new_state = apply_action(state, action)  # 执行行动得到新状态
            score = minimax(new_state, is_max_player=False)  # 切换到Min玩家
            max_score = max(max_score, score)  # 最大化收益
        return max_score
    
    # 3. Min玩家回合：选择所有子状态的最小值
    else:
        min_score = float('inf')
        for action in get_all_actions(state):
            new_state = apply_action(state, action)
            score = minimax(new_state, is_max_player=True)  # 切换到Max玩家
            min_score = min(min_score, score)  # 最小化Max的收益
        return min_score
```
#### 关键函数说明 
- `is_terminal(state)`：判断当前状态是否为终局（如取数游戏中数组为空）； 
- `evaluate(state)`：评估终局对Max的收益（如取数游戏中Max与Min的得分差）； 
- `get_all_actions(state)`：获取当前状态下所有可能的行动（如取左/取右）； 
- `apply_action(state, action)`：执行行动，返回新状态（如取左后剩余的数组）。 
### 五、与上一题“取数游戏”的关联 
上一题的“得分差递归”本质是 **Minimax的简化版**： 
- 用“得分差”直接替代“evaluate(state)”（终局得分差=Max的收益）； 
- 省略了显式的“Max/Min节点判断”，通过 `nums[left] - dfs(...)` 隐含实现：  
- `dfs(left+1, right)` 本质是Min玩家的最优收益（最小化Max的得分差）；  
- `nums[left] - 得分差` 等价于Max玩家在当前行动后的净收益。 
### 六、优化：Alpha-Beta剪枝（可选） 
原始Minimax会遍历所有可能的子状态，效率较低。**Alpha-Beta剪枝** 可通过“提前放弃无意义的分支”优化效率： 
- `alpha`：Max玩家当前能保证的最大收益（低于alpha的分支无需考虑）； 

- `beta`：Min玩家当前能保证的最小收益（高于beta的分支无需考虑）； 

- 当 `alpha ≥ beta` 时，当前分支对最终结果无影响，直接剪枝。 优化后的框架： 

```python
  def minimax_alpha_beta(state, is_max_player, alpha, beta):
    if is_terminal(state):
        return evaluate(state)

    if is_max_player:
        max_score = -float('inf')
        for action in get_all_actions(state):
            new_state = apply_action(state, action)
            score = minimax_alpha_beta(new_state, False, alpha, beta)
            max_score = max(max_score, score)
            alpha = max(alpha, max_score)  # 更新Max的最小保证收益
            if alpha >= beta:  # 剪枝：Min不会选择比beta更大的收益
                break
        return max_score
    else:
        min_score = float('inf')
        for action in get_all_actions(state):
            new_state = apply_action(state, action)
            score = minimax_alpha_beta(new_state, True, alpha, beta)
            min_score = min(min_score, score)
            beta = min(beta, min_score)  # 更新Min的最大保证收益
            if alpha >= beta:  # 剪枝：Max不会选择比alpha更小的收益
                break
        return min_score
```
### 七、适用场景与核心特点 
#### 适用场景 
- 双人零和博弈：棋类（井字棋、国际象棋）、取数游戏、石头剪刀布进阶版等； 
- 状态空间可枚举：每个状态的可能行动有限（否则递归深度过深，需结合其他优化）。 
#### 核心特点 
- 完全信息：双方都知道所有状态和行动结果（如棋类，无隐藏信息）； 
- 最优策略：假设对手会采取对自己最不利的行动，从而推导自己的最优选择； 
- 递归本质：通过“状态抽象+轮流切换玩家”，将复杂博弈转化为子问题的最优解。 
### 总结 Minimax算法的核心是“**换位思考+最优子问题**”： 
- Max玩家：最大化自己的最小收益； 
- Min玩家：最小化Max的最大收益； 
- 上一题的取数游戏是Minimax的简化应用，用“得分差”直接建模收益，省略了显式的玩家切换逻辑。 如果遇到“双人轮流决策、均最优、零和”的问题，直接套用Minimax框架（或简化版）即可解决。

## 题目

**初阶**

[洛谷 P1247 取数游戏](https://www.luogu.com.cn/problem/P1247)

[POJ 2348 Euclid's Game（简单博弈 + Minimax 思想）](http://poj.org/problem?id=2348)

[HDU 1846 Brave Game（巴什博弈，Minimax 简化版）](http://acm.hdu.edu.cn/showproblem.php?pid=1846)

**进阶**

[洛谷 P1084 疫情控制](https://www.luogu.com.cn/problem/P1084)

[Codeforces Round #102 (Div. 2) D. Game with Powers](https://codeforces.com/contest/143/problem/D)

[POJ 3734 Blocks](http://poj.org/problem?id=3734)

**高阶**

[洛谷 P4363 [九省联考 2018] 一双木棋 chess](https://www.luogu.com.cn/problem/P4363)

[HDU 3037 Saving Beans](http://acm.hdu.edu.cn/showproblem.php?pid=3037)
