# design-pattern
设计模式
## day17 建造者模式

定义：

*   拥有相同的物料，但是组合方式不同产生不同的结果，就是建造者模式需要解决的问题

### 实例：不同的装修风格，采用不同的物料组合

![image.png](https://note.youdao.com/yws/res/9/WEBRESOURCEab43e91953ce7502c345157be37f0fe9)

吊顶、涂料、地板、瓷砖都有不同的品牌可以选择

**ifelse实现**

工程目录

![image.png](https://note.youdao.com/yws/res/0/WEBRESOURCE8017354a1def9e0735af2628216bee90)

只有一个类，几千行代码的实现：

```java
public class DecorationPackageController {

    public String getMatterList(BigDecimal area, Integer level) {

        List<Matter> list = new ArrayList<Matter>(); // 装修清单
        BigDecimal price = BigDecimal.ZERO;          // 装修价格

        // 豪华欧式
        if (1 == level) {

            LevelTwoCeiling levelTwoCeiling = new LevelTwoCeiling(); // 吊顶，二级顶
            DuluxCoat duluxCoat = new DuluxCoat();                   // 涂料，多乐士
            ShengXiangFloor shengXiangFloor = new ShengXiangFloor(); // 地板，圣象

            list.add(levelTwoCeiling);
            list.add(duluxCoat);
            list.add(shengXiangFloor);

            price = price.add(area.multiply(new BigDecimal("0.2")).multiply(levelTwoCeiling.price()));
            price = price.add(area.multiply(new BigDecimal("1.4")).multiply(duluxCoat.price()));
            price = price.add(area.multiply(shengXiangFloor.price()));

        }

        // 轻奢田园
        if (2 == level) {

            LevelTwoCeiling levelTwoCeiling = new LevelTwoCeiling(); // 吊顶，二级顶
            LiBangCoat liBangCoat = new LiBangCoat();                // 涂料，立邦
            MarcoPoloTile marcoPoloTile = new MarcoPoloTile();       // 地砖，马可波罗

            list.add(levelTwoCeiling);
            list.add(liBangCoat);
            list.add(marcoPoloTile);

            price = price.add(area.multiply(new BigDecimal("0.2")).multiply(levelTwoCeiling.price()));
            price = price.add(area.multiply(new BigDecimal("1.4")).multiply(liBangCoat.price()));
            price = price.add(area.multiply(marcoPoloTile.price()));

        }

// 省略...
```

所有的代码都挤在一个类里面完成，后面需要更改物料，或者是添加新的组合需要改动的地方很多。不满足单一职责，可复用的原则。

**建造者模式实现**

工程结构：

![image.png](https://note.youdao.com/yws/res/c/WEBRESOURCE448cc9467940d5ead0220841a13c566c)

IMenu、DecorationPackageMenu、Builder三个类分别为包装接口类、包装接口实现类、组装实现类。

创建者模型结构如下：

![image.png](https://note.youdao.com/yws/res/4/WEBRESOURCE23769a5797df231fd3d43f8414922864)

```java
public interface IMenu {

    /**
     * 吊顶
     */
    IMenu appendCeiling(Matter matter);

    /**
     * 涂料
     */
    IMenu appendCoat(Matter matter);

    /**
     * 地板
     */
    IMenu appendFloor(Matter matter);

    /**
     * 地砖
     */
    IMenu appendTile(Matter matter);

    /**
     * 明细
     */
    String getDetail();

}
```

IMenu定义了组合的方法：添加吊顶、涂料、地板、地砖，返回具体信息

```java
public class DecorationPackageMenu implements IMenu {

    private List<Matter> list = new ArrayList<Matter>();  // 装修清单
    private BigDecimal price = BigDecimal.ZERO;      // 装修价格

    private BigDecimal area;  // 面积
    private String grade;     // 装修等级；豪华欧式、轻奢田园、现代简约

    private DecorationPackageMenu() {
    }

    public DecorationPackageMenu(Double area, String grade) {
        this.area = new BigDecimal(area);
        this.grade = grade;
    }

    public IMenu appendCeiling(Matter matter) {
        list.add(matter);
        price = price.add(area.multiply(new BigDecimal("0.2")).multiply(matter.price()));
        return this;
    }

    public IMenu appendCoat(Matter matter) {
        list.add(matter);
        price = price.add(area.multiply(new BigDecimal("1.4")).multiply(matter.price()));
        return this;
    }

    public IMenu appendFloor(Matter matter) {
        list.add(matter);
        price = price.add(area.multiply(matter.price()));
        return this;
    }

    public IMenu appendTile(Matter matter) {
        list.add(matter);
        price = price.add(area.multiply(matter.price()));
        return this;
    }

    public String getDetail() {

        StringBuilder detail = new StringBuilder("\r\n-------------------------------------------------------\r\n" +
                "装修清单" + "\r\n" +
                "套餐等级：" + grade + "\r\n" +
                "套餐价格：" + price.setScale(2, BigDecimal.ROUND_HALF_UP) + " 元\r\n" +
                "房屋面积：" + area.doubleValue() + " 平米\r\n" +
                "材料清单：\r\n");

        for (Matter matter: list) {
            detail.append(matter.scene()).append("：").append(matter.brand()).append("、").append(matter.model()).append("、平米价格：").append(matter.price()).append(" 元。\n");
        }

        return detail.toString();
    }

}
```

DecorationPackageMenu 是接口的具体实现类

```java
public class Builder {

    public IMenu levelOne(Double area) {
        return new DecorationPackageMenu(area, "豪华欧式")
                .appendCeiling(new LevelTwoCeiling())    // 吊顶，二级顶
                .appendCoat(new DuluxCoat())             // 涂料，多乐士
                .appendFloor(new ShengXiangFloor());     // 地板，圣象
    }

    public IMenu levelTwo(Double area){
        return new DecorationPackageMenu(area, "轻奢田园")
                .appendCeiling(new LevelTwoCeiling())   // 吊顶，二级顶
                .appendCoat(new LiBangCoat())           // 涂料，立邦
                .appendTile(new MarcoPoloTile());       // 地砖，马可波罗
    }

    public IMenu levelThree(Double area){
        return new DecorationPackageMenu(area, "现代简约")
                .appendCeiling(new LevelOneCeiling())   // 吊顶，二级顶
                .appendCoat(new LiBangCoat())           // 涂料，立邦
                .appendTile(new DongPengTile());        // 地砖，东鹏
    }

}
```

Builder 则是具体的组合类，定义了不同的装修风格

后续的具体测试类：

```java
System.out.print(new Builder.levelOne(13.5).getDetail())
```

如果基本的物料不会变，但是组合模式经常发生变化，这可以使用建造者模式来实现，通过包装接口来实现物料的装配，通过建造者来实现具体的组合，后面更改组合的变化则只需要更改建造者类即可。

## day16 抽象工厂模式

解决idea出现 Solved Cannot resolve symbol [原文链接](https://taogenjia.com/2023/02/20/How-to-Solve-The-Problem-Cannot-resolve-symbol-in-Intellij-IDEA/):&#x20;

*   出现这个原因是因为idea的缓存出问题了，将缓存清理一下：File -> Invalidate Caches -> checked “clear file system cache and Local History”, and then click “Invalidate and Restart”.

定义：

*   抽象工厂模式和工厂方法模式类似，也是为了解决接口选择的问题，但是抽象⼯⼚是⼀个中⼼⼯⼚，用于创建其他⼯⼚。

例子：

*   一个软件可能在不同的操作系统上面运行，不同的操作系统都有菜单、鼠标移动、键盘输入等功能，但是这些功能的实现逻辑都不一样。比如windows、linux、mac的操作系统的回车分别是\n,\n\r,\r。抽象工厂就是用于创建windows、linux、mac三种不同的工厂的设计模式。

### 实例：Redis服务的升级，从单机的RedisUtil升级成集群EGM和IIR

**ifelse方式**

代码结构：

![image.png](https://note.youdao.com/yws/res/b/WEBRESOURCE6f335f69e566510cd7f4f4e6d79ca44b)

```java
public class CacheServiceImpl implements CacheService {

    private RedisUtils redisUtils = new RedisUtils();

    private EGM egm = new EGM();

    private IIR iir = new IIR();

    public String get(String key, int redisType) {

        if (1 == redisType) {
            return egm.gain(key);
        }

        if (2 == redisType) {
            return iir.get(key);
        }

        return redisUtils.get(key);
    }

    public void set(String key, String value, int redisType) {

        if (1 == redisType) {
            egm.set(key, value);
            return;
        }

        if (2 == redisType) {
            iir.set(key, value);
            return;
        }

        redisUtils.set(key, value);
    }
// 省略...
}
```

这样子后面需要拓展的时候非常头疼，需要修改原来的代码，并且需要修改每一个方法，修改的时候容易导致原来的功能不可用。

**抽象工厂模式**

工程结构：

![image.png](https://note.youdao.com/yws/res/5/WEBRESOURCEb7026f454dfe20fd00d28bdeccc6e175)

结构：

![image.png](https://note.youdao.com/yws/res/1/WEBRESOURCE26e77171785de36751bbae0e947139b1)

ICacheAdapter是适配器接口，因为集群A和集群B的方法名称都不相同，因此需要有一个适配器来适配方法。JDKProxy，JDKInvocationHandler是代理类的实现。通过传入不同的CacheAdapter来完成不同集群的接入。

```java
public class EGMCacheAdapter implements ICacheAdapter {

    private EGM egm = new EGM();

    public String get(String key) {
        return egm.gain(key);
    }

    public void set(String key, String value) {
        egm.set(key, value);
    }

    public void set(String key, String value, long timeout, TimeUnit timeUnit) {
        egm.setEx(key, value, timeout, timeUnit);
    }

    public void del(String key) {
        egm.delete(key);
    }
}
```

调用：

```java
public void test_CacheService() throws Exception {

    CacheService proxy_EGM = JDKProxy.getProxy(CacheServiceImpl.class, new EGMCacheAdapter());
    proxy_EGM.set("user_name_01", "小傅哥");
    String val01 = proxy_EGM.get("user_name_01");
    System.out.println("测试结果：" + val01);

    CacheService proxy_IIR = JDKProxy.getProxy(CacheServiceImpl.class, new IIRCacheAdapter());
    proxy_IIR.set("user_name_01", "小傅哥");
    String val02 = proxy_IIR.get("user_name_01");
    System.out.println("测试结果：" + val02);
}
```



## day15 工厂方法模式

定义：

*   创建型设计模式，其在父类中提供一个创建对象的方法，允许子类决定实例化对象的类型。

优点：

*   增加代码的拓展性、可读性
*   屏蔽每一个功能类中的具体实现逻辑，外部只需要知道如何调用即可
*   可以去掉众多的ifelse语句，但是需要实现的类非常多，每个类只关注自身功能的实现（满足单一职责）
*   减少了代码的耦合，无需更改调用方法就可以在程序中引入新的产品类型（满足开闭原则）

缺点：

*   如果新商品类型很多，那么实现的子类会极速扩张

### 实例

eg：模拟发奖多种商品，优惠券、实物商品、第三方爱奇艺兑换卡

![image.png](https://note.youdao.com/yws/res/29226/WEBRESOURCE419baf54f726ac9c1bf900b5cbe93d00)

#### ifelse方式

工程结构：

![image.png](https://note.youdao.com/yws/res/29234/WEBRESOURCE37a348c38a54f717ccd87083a4c72c83)

*   AwardReq 为入参对象、AwardRes为出参对象、PrizeController为接口类

```java
public class PrizeController {

    private Logger logger = LoggerFactory.getLogger(PrizeController.class);

    public AwardRes awardToUser(AwardReq req) {
        String reqJson = JSON.toJSONString(req);
        AwardRes awardRes = null;
        try {
			if (req.getAwardType() == 1) {
				// 具体业务1
			}
			else if(req.getAwardType() == 2) {
				// 具体业务2
			} 
			else if(req.getAwardType() == 3) {
				// 具体业务3
			}
		}
	}
}
```

*   可以看到所有的业务都写在了一起，并且后期拓展时，增加商品类型需要先阅读这一大坨代码，让人很痛苦

#### 使用工厂方法模式

![image.png](https://note.youdao.com/yws/res/29249/WEBRESOURCEa2561a5360d7d6480833d2812bff02ba)

ICommodity为接口父类，其定义了一个公共的方法

```java
public interface ICommodity {

    void sendCommodity(String uId, String commodityId, String bizId, Map<String, String> extMap) throws Exception;

}
```

CardCommodityService.java、CouponCommodityService.java、GoodsCommodityService.java为优惠券、实体商品、爱奇艺的具体实现类

StoreFactory.java为商店工厂，其按照类型实现各种商品的服务。可以非常干净地处理你的代码。

```java
public class StoreFactory {

    public ICommodity getCommodityService(Integer commodityType) {
        if (null == commodityType) return null;
        if (1 == commodityType) return new CouponCommodityService();
        if (2 == commodityType) return new GoodsCommodityService();
        if (3 == commodityType) return new CardCommodityService();
        throw new RuntimeException("不存在的商品服务类型");
    }

}
```

