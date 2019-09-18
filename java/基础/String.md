## [String 和 new String()的区别](https://www.cnblogs.com/Seeasunnyday/p/9087958.html)

String 和 new String()的区别

For Example

String str1 = "ABC"

String str2 = new String("ABC");

 

String str1 = "ABC"; 可能创建一个对象或者不创建对象。

如果"ABC" 这个字符串z在java String池中不存在，会在java String池中创建一个String str1= "ABC"的对象。然后把str1指向这个内存地址。之后用这种方式创建多少个值为"ABC"的字符串对象。始终只有一个内存地址被分配，之后都是String的copy。这种被称为‘字符串驻留’，所有的字符串都会在编译之后自动驻留。

 

String str2 = new String("ABC"); 至少会创建一个对象，也可能2个。

因为用到了new的关键字，肯定会在heap中创建一个str2的对象。它的value值是"ABC"，同时如果这个字符串在string池中不存在，会在string词中创建这个string对象"ABC"。

 

String str1 = new String("ABC");

String str2 = new String("ABC");

System.out.println(str1.equals(str2)); // true 比较的值

System.out.println(str1 == str2); // false 比较的是内存地址。

 

String str3 = "ABC"

String str4 = "ABC"

String str5 = "AB"+"C";

System.out.println(str3 == str4); //true 在string池中都是一个内存地址被分配给str3,str4,str5

System.out.println(str3 == str5); //true

 

String str6 = "AB";

String str7 = str6 + "C";

System.out.println(str3 == str7); //false

 

str6在编译的时候已经确认为string池的对象。

str7在编译的时候不能确认，故str7是一个引用变量。

str6+"C"的过程是创建了一个StringBuffer对象，然后用StringBuffer对象执行append方法追加，最后再转成String类型，也就是str7是放在heap里面的对象，str6是放在String常量池里的。两个的内存地址不一样。故结果为false。