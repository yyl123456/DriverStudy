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




