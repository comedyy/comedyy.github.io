### 设计模式

总结几个比较常用的游戏中的设计模式

使用了这些设计模式的代码，看起来都很简洁。使用设计模式可以简化代码复杂度。



#### 抽象工厂

抽象 `对象获取` 

`AbstractFactory` 跟普通的工厂不是很一样，就它可以由很多 派生的工厂，用于生成不同的东西。

比如 两个军工厂，一个是海军工厂，一个是陆军工厂，给他们一个订单，生产一条裤子，海军生产的是蓝色的裤子，而陆军则是绿色。

这样你就可以给 目标军队装备不同的军工厂，来达到穿不同款式的衣服的目的。

就军队不用关心是哪个工厂给他做衣服，它只管收衣服就行了。



#### Builder

抽象 `构建对象`

对象通过对应的Builder来创建。

Builder是个黑盒，外部不关心它的具体创建过程，只要调用它的创建函数，就可以获得对应的对象。

应用： `protobuf` 用它来生产一个消息包。



#### Command

抽象 `行为`

Command是个基类，它包含 `Excute`函数。所有的Command都有它自己的`Excute`实现。

而外部不需要知道它里面具体实现，只管调用就行。



应用： 遥控器



#### Decorator

被修饰对象 A， Decorator 对象这是继承自 A， 它跟A有同样的接口。通过修饰器来修改A对象的行为。

不同于直接继承，修饰器修饰的是现有的对象，而不是生成新对象，只是它跟被修饰者A有共同的接口。

```c#
   abstract class MeleeDecorator : MeleeAttack
    {
        MeleeAttack meleeAttack;

        public override int Attack() 			// 基础修饰器实现了被修饰对象的功能，通过继承MeleeDecorator来修改Attack的行为。
        {
            if (meleeAttack != null)
                return meleeAttack.Attack();
            else
                return 0;
        }

        public void SetMeleeAttack(MeleeAttack meleeAttack)
        {
            this.meleeAttack = meleeAttack;
        }
    }
    
        baseMeleeAttack = new BaseMelee(); // 被修饰对象

        hardPunch = new HardPunch();		// 修饰器1
        hardPunch.SetMeleeAttack(baseMeleeAttack);

        kiPunch = new KiPunch(kiParticles);// 修饰器2
        kiPunch.SetMeleeAttack(baseMeleeAttack);

        // Set up the Super Ki Punch
        HardPunch hpDeco = new HardPunch();
        hpDeco.SetMeleeAttack(baseMeleeAttack);
        superKiPunch = hpDeco;
        KiPunch kpDeco = new KiPunch(kiParticles);// 修饰器3
        kpDeco.SetMeleeAttack(superKiPunch);
        superKiPunch = kpDeco;
```

比如 一把枪，装上了 瞄准器 ==》 Attack的精度提高， 装上了好弹夹==》可以Attack了更久， 装上了 加速器==》 可以打更快。

应用：行为树的Decorator节点。



#### Observer

灵活的事件监听。



#### Strategy

抽象 `行为`

比如有 10中走路方式，我需要把它分配给10个角色，我们的基类可以有一个`BaseWalk`对象，这样`BaseWalk`可以装配成不同的Walk对象。

然而这10个对象可能并不是都有共同的基类。他们只要有共同的接口，使用strategy模式。



新建行为对象，把目标对象往行为对象里面装，然后通过调用这个行为对象来控制这个目标对象。

跟Decorator有点像，都是把目标对象往自己身上塞，然后通过调用自己来控制目标的行为。



如果当前行为只是其中一个模块使用，就可以用它来横向扩展，减少往 目标对象里面添加的代码量。