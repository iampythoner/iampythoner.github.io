---
layout: cnblog_post
title:  "简单工厂、工厂方法、抽象工厂"
date:   2017-09-15 23:34:39
categories: 设计模式
---
### 简单工厂 并不是GoF 23中设计模式之一

```python
# role: abstract product
class AbstractTV(object):
    def play(self):
        pass

# role: concrete product
class HaierTV(AbstractTV):
    def play(self):
        print("Haier TV playing")

class SonyTV(AbstractTV):
    def play(self):
        print("Sony TV playing")

# role: factory
class TVFactory(object):
    @staticmethod
    def createTV(type_name):
        abstractTV = None
        if type_name == "Haier":
            abstractTV = HaierTV()
        elif type_name == "Sony":
            abstractTV = SonyTV()
        return abstractTV

# test
def main():
    tv = TVFactory.createTV("Haier")
    tv.play()

if __name__ == '__main__':
    main()
```

缺点：<br/>
产品类和工厂类耦合，一旦添加产品就不得不修改工厂逻辑。如果新增具体产品，至少修改三处：<br/>
1.添加新产品类<br/>
2.还要修改生产产品的工厂类代码<br/>
3.调用工厂类处要传递新的产品标识<br/>

### 工厂方法

```python
# role: abstract product
class AbstractTV(object):
    def play(self):
        pass

# role: concrete product
class HaierTV(AbstractTV):
    def play(self):
        print("Haier TV playing")

class SonyTV(AbstractTV):
    def play(self):
        print("Sony TV playing")

# new role: abstract factory
class AbstractTVFactory(object):
    def createTV(self):
        pass

# role: concrete factory
class HaierTVFactory(AbstractTVFactory):
    def createTV(self):
        return HaierTV()

class SonyTVFactory(AbstractTVFactory):
    def createTV(self):
        return SonyTV()

# test
def main():
    factory_class_type = HaierTVFactory
    # factory_class_type = SonyTVFactory

    tv = factory_class_type().createTV()
    tv.play()

if __name__ == '__main__':
    main()
```

工厂方法的优缺点：<br/><br/>
优点：<br/>
1.在工厂方法模式中，工厂方法用来创建客户所需要的产品，同时还向客户(使用产品的地方)隐藏了哪种具体产品类将被实例化这一细节，
用户只需要关心所需产品对应的工厂，无需关心创建细节，甚至无需知道具体产品类的类名。<br/>
2.基于工厂角色和产品角色的多态性设计是工厂方法模式的关键。它能够使工厂可以自主确定创建何种产品对象，而如何创建这个
对象的细节完全封装在具体工厂内部。工厂方法模式之所以又被称为多态工厂模式，是因为所有具体工厂类都具有同一抽象父类。<br/>
3.使用工厂方法模式的另一个有点是在系统中加入新产品时，无需修改抽象工厂和抽象产品提供的接口，无需修改客户端(使用产品的地方)，
也无需修改其他的具体工厂和具体产品，而只要添加一个具体工厂和具体产品就可以了。这样，系统的可扩展性也就变得非常好，完全符合"开闭原则"。<br/><br/>
缺点：<br/>
如果生产多个产品，那么系统复杂度大大提升：<br/>
1.创建新增产品的抽象类和具体类<br/>
2.每个产品要创建该产品相应的具体工厂、抽象工厂<br/><br/>
对于1的创建时必须的<br/>
对于2可以进行适当的优化,对相同的厂商的产品在一个工厂生产，即一个工厂可以生产多个产品，这样就演变出来了抽象工厂<br/>

### 抽象工厂

```python
# role: abstract product
class AbstractTV(object):
    def play(self):
        pass

class AbstractComputer(object):
    def typewrite(self):
        pass

# role: concrete product
# TV
class HaierTV(AbstractTV):
    def play(self):
        print("Haier TV playing")

class SonyTV(AbstractTV):
    def play(self):
        print("Sony TV playing")

# Computer
class HaierComputer(AbstractComputer):
    def typewrite(self):
        print("Haier computer typewriting")

class SonyComputer(AbstractComputer):
    def typewrite(self):
        print("Sony computer typewriting")
```

```python
# role: abstract factory
class AbstractAppliancesFactory(object):
    def createTV(self):
        pass

    def createComputer(self):
        pass

# role: concrete factory
class HaierAppliancesFactory(object):
    def createTV(self):
        return HaierTV()

    def createComputer(self):
        return HaierComputer()

class SonyAppliancesFactory(object):
    def createTV(self):
        return SonyTV()

    def createComputer(self):
        return SonyComputer()
```

```python
# test
def main():
    appliances_factory_class_type = HaierAppliancesFactory
    # appliances_factory_class_type = SonyAppliancesFactory

    appliances_factory = appliances_factory_class_type()
    tv = appliances_factory.createTV()
    tv.play()
    computer = appliances_factory.createComputer()
    computer.typewrite()

if __name__ == '__main__':
    main()
```
