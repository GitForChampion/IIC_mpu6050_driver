 # IIC_mpu6050_driver
I2C总线二级外设驱动开发之设备树匹配

### MPU6050
三轴角速度+三轴加速度+温度传感器

![引脚描述图](https://img-blog.csdnimg.cn/2a1021e931444a198012b15b2680ad24.png)

```c
#define SMPLRT_DIV  0x19 //陀螺仪采样率，典型值：0x07(125Hz)
#define CONFIG   0x1A //低通滤波频率，典型值：0x06(5Hz)
#define GYRO_CONFIG  0x1B //陀螺仪自检及测量范围，典型值：0xF8(不自检，+/-2000deg/s)
#define ACCEL_CONFIG 0x1C //加速计自检、测量范围，典型值：0x19(不自检，+/-G)
#define ACCEL_XOUT_H 0x3B
#define ACCEL_XOUT_L 0x3C
#define ACCEL_YOUT_H 0x3D
#define ACCEL_YOUT_L 0x3E
#define ACCEL_ZOUT_H 0x3F
#define ACCEL_ZOUT_L 0x40
#define TEMP_OUT_H  0x41
#define TEMP_OUT_L  0x42
#define GYRO_XOUT_H  0x43
#define GYRO_XOUT_L  0x44
#define GYRO_YOUT_H  0x45
#define GYRO_YOUT_L  0x46
#define GYRO_ZOUT_H  0x47
#define GYRO_ZOUT_L  0x48
#define PWR_MGMT_1  0x6B //电源管理，典型值：0x00(正常启用)
```

# 设备树匹配
设备树匹配：内核启动时根据设备树自动产生的设备 ------ 优先级最高

注意事项：
1. 无需编写device模块，只需编写driver模块
2. 使用compatible属性进行匹配，注意设备树中compatible属性值不要包含空白字符
3. id_table可不设置，但struct platform_driver成员driver的name成员必须设置

```c
/*platform driver框架*/
#include <linux/module.h> 
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/platform_device.h>

static int driver_probe(struct platform_device *dev)
{
	printk("platform: match ok!\n");
	return 0;
}

static int driver_remove(struct platform_device *dev)
{
	printk("platform: driver remove\n");
	return 0;
}

struct platform_device_id testdrv_ids[] = 
{
		[0] = {.name = "test_device"},
    [1] = {.name = "abcxyz"},
    [2] = {}, //means ending
};

struct of_device_id test_of_ids[] = 
{
		[0] = {.compatible = "xyz,abc"},
    [1] = {.compatible = "qwe,opq"},
    [2] = {},
};

struct platform_driver test_driver = {
	.probe = driver_probe,
	.remove = driver_remove,
	.driver = {
	.name = "xxxxx", //必须初始化
        .of_match_table = test_of_ids,
	},
};

static int __init platform_driver_init(void)
{
	platform_driver_register(&test_driver);
	return 0;
}

static void __exit platform_driver_exit(void)
{
	platform_driver_unregister(&test_driver);
}

module_init(platform_driver_init);
module_exit(platform_driver_exit);
MODULE_LICENSE("Dual BSD/GPL");
```

![](https://img-blog.csdnimg.cn/8e6c1c70c5e8410882097d9a5db44dc3.png)
