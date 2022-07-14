# Java中的比较器（Comparable和Comparator）

## 为什么要用比较器？

在java中，普通情况下，只能使用==，!=，equals来比较对象，不能使用>或<

但是有时候我们会涉及到 通过对象中的某个属性来比较对象的大小，这时候比较器的作用就凸显出来了，（当然，自己想写排序算法也没啥问题，。。。）

## Comparable接口

自然排序 是在包装类里实现Comparable接口，然后重写compareTo(obj)方法（比较两个对象大小的方式）

使用步骤：在类中实现Comparable接口 -> 重写compareTo(obj)方法 -> 在compareTo(obj)方法中知名如何排序

```java
重写compareTo(obj)的规则：
	如果当前对象this大于形参对象obj，则返回正整数
	如果当前对象this小于形参对象obj，则返回负整数
	如果当前对象this等于形参对象obj，则返回0
	
默认是从小到大排序，想从大到小排序可以把上述规则反过来就行了。
```

```java
public class Goods implements  Comparable{

    private String name;
    private double price;

    //指明商品比较大小的方式:照价格从低到高排序,再照产品名称从高到低排序
    @Override
    public int compareTo(Object o) {
        if(o instanceof Goods){
            Goods goods = (Goods)o;
            //从低到高
            if(this.price > goods.price){
                return 1;
            }else if(this.price < goods.price){
                return -1;
            }else{
               //从高到低 逆规则
               return -this.name.compareTo(goods.name);
            }
        }

        throw new RuntimeException("传入的数据类型不一致！");
    }

}
```

## Comparator接口

定制排序

使用步骤：创建比较器 -> 在指定方法（数组排序，Collections集合中，TreeSet的构造方法）中使用

指定方法：

1. Arrays.sort(goods,com);

				   2. Collections.sort(coll,com);
				   2. new TreeSet(com);

```java
重写compare(Object o1,Object o2)方法，比较o1和o2的大小：
	如果o1>o2 则返回正整数
	如果o1<o2 则返回负整数
	如果o1==o2 则返回0

默认是从小到大排序，想从大到小排序可以把上述规则反过来就行了。
```

```java
Comparator com = new Comparator() {
    //指明商品比较大小的方式:照产品名称从低到高排序,再照价格从高到低排序
    @Override
    public int compare(Object o1, Object o2) {
        if(o1 instanceof Goods && o2 instanceof Goods){
            Goods g1 = (Goods)o1;
            Goods g2 = (Goods)o2;
            if(g1.getName().equals(g2.getName())){
                return -Double.compare(g1.getPrice(),g2.getPrice());
            }else{
                return g1.getName().compareTo(g2.getName());
            }
        }
        throw new RuntimeException("输入的数据类型不一致");
    }
}

```

## Comparable和Comparator的不同

Comparable接口的方式使用场景：是在实体类中重写接口的方法，该方式是固定的，优点是保证了在任何位置都可以比较大小，缺点是比较方式固定，不灵活

Comparator接口的方式使用场景：是在想进行比较的时候创建比较器，优点是比较方式很灵活，缺点是每次想比较的时候都要写。