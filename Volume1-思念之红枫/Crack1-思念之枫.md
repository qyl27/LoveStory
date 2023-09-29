## 解析
我们定义初始状态为O，进行旋转的操作为T，其中T中每一个元素表示的是O矩阵在对应位置的旋转操作，这样做的理由有以下几条：

* T中每一个元素满足结合律，比如在 $T[0, 0]$ 处旋转了+2之后又旋转了-1，其最终结果等价于旋转了+1
* T中每一个元素满足交换律，也就是说先执行 $T[0, 0]$ 旋转后执行 $T[0, 1]$ 旋转和先执行 $T[0, 1]$ 旋转后执行 $T[0, 0]$ 旋转是等价的，因此我们可以按顺序执行T中的每一次旋转

T在[i, j]处的计算公式为：

$T[i, j] = -O[i, j] + T[i - 1, j] + T[i - 1, j] + T[i, j - 1] + T[i, j + 1] + 2$

以4x4的矩阵为例，现在只需给出矩阵的第一行的状态，将上面的公式变一下形，计算出第二行（当然第三行和第四行也同理）：

$T[1, j] = O[0, j] + T[0, j] - T[1, j] - T[0, j - 1] - T[0, j + 1] - 2$

由于第一、第二和第三行都不是最后一行，因此我们不必担心其左右的扩散会扰乱其最终状态，这些都由下一行在上一行的扩散作用下得到修正。只有最后一行没有下一行，所以最后一行还需要保证其左右的扩散作用恰好得到目标矩阵。在大多数情况下，这是不成立的，只有当成立时才存在解。因此，对于一个NxN的矩阵，我们只需要遍历 $4^N$ 次来寻找其最优解。

## 参考实现
```c++
#include <iostream>
#include <vector>
#include <algorithm>

struct Matrix
{
    size_t size;
    std::vector<int> items;
    
    Matrix(size_t size) : size(size), items(pow(size, 2), 0) {}
    
    int get(int row, int col)
    {
        if(row >= 0 && row < size && col >= 0 && col < size)
        {
            return items[row * size + col]; 
        }
        else
        {
            return 0;
        }
    }
    
    void set(int row, int col, int val)
    {
        items[row * size + col] = val;
    }
};

/** map an integer into {-1, 0, 1, 2} */
int map_into(int op) {
    op %= 4;
    if(op == 3)
    {
        op = -1;
    }
    else if(op == -3)
    {
        op = 1;
    }
    else if(op == -2){
        op = 2;
    }
    return op;
}

bool check(Matrix& origin, Matrix& op)
{
    for(int row = 1; row < origin.size - 1; row++)
    {
        for(int col = 0; col < origin.size; col++)
        {
            op.set(row, col, map_into(
                origin.get(row - 1, col) + op.get(row - 1, col)
                - op.get(row - 1, col - 1) - op.get(row - 1, col + 1)
                - op.get(row - 2, col) - 2
            ));
        }
    }
    
    int last_row = origin.size - 1;
    for(int col = 0; col < origin.size; col++)
    {
        op.set(last_row, col, map_into(
            origin.get(last_row - 1, col) + op.get(last_row - 1, col)
            - op.get(last_row - 1, col - 1) - op.get(last_row - 1, col + 1)
            - op.get(last_row - 2, col) - 2
        ));
        
        if(col > 0 && 
                map_into(origin.get(last_row, col - 1) + op.get(last_row, col - 1)
                                 - op.get(last_row, col) - op.get(last_row, col - 2)
                                 - op.get(last_row - 1, col - 1)) != 2)
        {
            return false; 
        }
    }
    
    return true;
}

int get_steps(Matrix& op)
{
    int sum = 0;
    for(int row = 0; row < op.size; row++)
    {
        for(int col = 0; col < op.size; col++)
        {
            sum += abs(op.get(row, col));
        }
    }
    return sum;
}

int main()
{
    int cols = 0;
    std::cin >> cols;
    
    Matrix origin(cols);

    for(int row = 0; row < cols; row++)
    {
        for(int col = 0; col < cols; col++)
        {
            int val = 0;
            std::cin >> val;
            origin.set(row, col, val);
        }

    }
    
    Matrix op(cols);
    /* set line 0 to -1 */
    for(int col = 0; col < cols; col++)
    {
        op.set(0, col, -1); 
    }
    
    int min_steps = -1;
    for(int i = 0; i < pow(4, cols); i++)
    {
        for(int col = 0; col < cols; col++)
        {
            if(op.get(0, col) > 2)
            {
                op.set(0, col, -1);
                op.set(0, col + 1, op.get(0, col + 1) + 1);
            }
        }
        
        if(check(origin, op))
        {
            int steps = get_steps(op);
            if(min_steps == -1 || steps < min_steps)
            {
                min_steps = steps; 
            }
        }
        
        op.set(0, 0, op.get(0, 0) + 1); 
    }
    
    if(min_steps != -1)
    {
        std::cout << min_steps << std::endl;
    }
    else
    {
        std::cout << "NO SOLUTION\n";
    }
    
    return 0;
}
```