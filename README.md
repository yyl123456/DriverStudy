usb_register(&my_usb_driver);  
static struct usb_driver my_usb_driver = {
    .name = "my_usb_driver",
    .id_table = my_usb_table,           //USB_VENDOR_ID, USB_PRODUCT_ID
    .probe = my_usb_probe,              //连上之后初始化，比如register chrdev、creat class&devices
    .disconnect = my_usb_disconnect,
};

my_usb_probe：
1 多device（/dev）支持、动态major
  alloc_chrdev_region(&dev_num, 0, 1, "my_device");    //major = MAJOR(dev_num);
  cdev_init(&my_cdev, &fops);
  my_cdev.owner = THIS_MODULE;                         //my_device通过THIS_MODULE 和 my_cdev与cdev_init关联
  cdev_add(&my_cdev, dev_num, 1);
  
  class_create
  device_create
2 单device（/dev）、不支持动态major
  major = register_chrdev(0, "my_device", &fops);
  
  class_create
  device_create


static struct file_operations fops = {
    .owner = THIS_MODULE,
    .open = my_open,
    .release = my_release,
    .read = my_read,
    .write = my_write,
    .unlocked_ioctl = my_ioctl,    //创建ioctl的字符设备的操作
};



多device（/dev）支持：
    struct my_device {
        struct cdev cdev;
        char data[100];  // 示例数据
    };
    static struct my_device *my_devices[NUM_DEVICES];
    for
        cdev_init(&my_devices[i]->cdev, &fops);
        my_devices[i]->cdev.owner = THIS_MODULE;
        cdev_add(&my_devices[i]->cdev, MKDEV(major, i), 1);

my_ioctl：
    struct my_device_data *data = file->private_data;
    switch (cmd) {
        case 0:
            data->mode = 1;
            printk(KERN_INFO "Switched to mode 1\n");
            break;
        case 1:
            data->mode = 2;
            printk(KERN_INFO "Switched to mode 2\n");
            break;
my_ioctl  op：
    # 切换到模式1
    ioctl /dev/my_device 0

    # 切换到模式2
    ioctl /dev/my_device 1
