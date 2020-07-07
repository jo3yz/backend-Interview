## 简介
- Substitution Failure Is Not An Error（匹配失败不是错误）
- 由于类型推断而引起的替换失败不会立刻引起编译错误，而是会进行一个类似于“编译期重载”的操作
- SFINAE使得C++可以在编译期进行一些方法的重载，这个是运行期0开销的，匹配失败的方法直接就不编译

## 需要注意的点
一旦类型匹配成功，模板就会被实例化，如果在模板函数内部发生了编译错误，**编译会失败！**，也就是说，**模板一旦实例化，SFINAE就无法让错误再被忽略了**

## 写一个is_class
```
template <typename T>
class is_class
{
private:
    template<typename U>
    static char helper(int U::*); // 只能指向int类型的U的成员变量

    template<typename U>
    static int helper(...);


public:
    constexpr static bool value = sizeof(helper<T>(nullptr)) == sizeof(char);
};
```