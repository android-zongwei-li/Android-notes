# 开发环境

1、Windows 默认有文件安全相关保护，会影响编译速度。

![image-20240511180500187](images/build提速/image-20240511180500187.png)

解决：将项目和Android sdk 等地址从保护路径中排除掉。

> Android Studio Hedgehog | 2023.1.1
>
> 这个版本可以在提示后自动配置，点击【Automatically】。

如果 AS 不能自动配，按如下步骤配置。

![image-20240511181228277](images/build提速/image-20240511181228277.png)

![image-20240511181358191](images/build提速/image-20240511181358191.png)

把 sdk 和项目路径填加进排除项：

![image-20240511181457576](images/build提速/image-20240511181457576.png)



