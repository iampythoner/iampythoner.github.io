---
layout: cnblog_post
title:  "简单工厂、工厂方法、抽象工厂"
date:   2017-09-15 23:34:39
categories: Python
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
产品类和工厂类耦合，如果新增具体产品，至少修改三处：<br/>
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


缺点<br/>
如果生产多个产品，那么系统复杂度大大提升：<br/>
1.创建新增产品的抽象类和具体类<br/>
2.每个产品要创建该产品相应的具体工厂、抽象工厂<br/><br/>
对于1的创建时必须的<br/>
对于2可以进行适当的优化,对相同的厂商的产品在一个工厂生产，即一个工厂可以生产多个产品<br/>

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
