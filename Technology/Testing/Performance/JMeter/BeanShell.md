# BeanShell

- 编写beanshell注意事项：
  - BeanShell里面不能使用JMeter内置函数，比如JMeter函数${__time(yyyyMMddHHmmss)}。在BeanShell里面可以用 new SimpleDateFormat("yyyyMMddHHmmss").format(new Date());代替。