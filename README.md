# Billcode
编码规则组件
# 编码规则组件-核心层

### 简介

> 在业务系统中，业务对象通常会有一个业务对象编号，业务对象编号作为业务对象具有业务意义的标识，一般由时间，序列号，常量等子段按照一定的规则拼接而成，
> 这个规则我们称为'编码规则'，各个子段我们称之为'编码元素'。考虑如下的业务对象编号：'TT-20150615-0001',编码规则为:
> （常量'TT-'+yyyyMMdd类型的时间'20150615'+常量'-'+序列号'0001'),如果时间变为'20150616',序列号重新从'0001'开始流水，那么我们称此
> 时间编码元素是'流水依据元素',所有的流水依据元素合起来叫做本编码规则的'流水依据'，同一个编码规则，可以有很多个流水依据！
> 
> 目前比较少编码规则的开源组件，网上能找到的也只是一些简单的实现类或者设计思路，
> 要么将规则写死在代码中，缺乏通用性和灵活性，要么缺少全面的考虑，本组件提供了一种通用的编码规则解决方案。核心层不关心编码规则的存储和获取,
> 由外部实现，以参数的形式传入编码规则，核心层负责根据编码规则进行业务对象编码的计算。


### 编码规则组件设计目标

- 尽量少依赖平台相关的jar包，以提供最大的可通用性
- 支持常见的编码元素，并提供扩展机制
- 编码规则可配置业务对象编码是否连续
- 编码规则支持按一些编码元素归零

### 编码规则组件模型说明

#### 一、编码规则模型    
编码规则组件根据编码规则生成业务对象编号，需要传入编码规则。  
核心层组件不关心编码规则的存储和获取，只关心编码规则的使用，定义了编码规则对象的模型接口，提供了编码规则的模型。  
编码规则对象`com.yonyou.uap.billcode.model.BillCodeRuleVO`。  
编码规则对象分为两部分信息:**编码规则基础信息**和**编码规则元素信息**  
> 编码规则基础信息模型：`com.yonyou.uap.billcode.model.IBillCodeBaseVO`  
> 编码规则元素信息模型：`com.yonyou.uap.billcode.model.IBillCodeElemVO`     


#### 二、业务对象封装模型
参见：
> `com.yonyou.uap.billcode.model.BillCodeBillVO`

#### 三、编码引擎运行时持久化对象
参见：
> `com.yonyou.uap.billcode.engine.persistence.vo.BillCodeReternVO` 
> `com.yonyou.uap.billcode.engine.persistence.vo.BillCodeSNVO`
> `com.yonyou.uap.billcode.engine.persistence.vo.PreCodeVO`

### 如何使用编码规则组件

通常情况下,编码元素可分为以下两种：  
**非业务对象信息**：系统时间，常量，序列号，随机码  
**业务对象信息**：业务对象上具体的值（包括：业务时间、字符、参照值（引用别的业务对象的id，code等））

#### 一、对于业务对象编号的编码规则仅仅需要非业务对象信息的情况

1. 实现编码规则引擎的持久化接口`com.yonyou.uap.billcode.engine.persistence.IBillCodeEngineService`

2. 实现编码规则引擎的锁接口 `com.yonyou.uap.billcode.lock.IBillCodeEngineLock`  
锁分为两种，数据库锁和内存锁，数据库锁只会调用lock方法，会在事务提交时自动解锁，内存锁会调用lock和unlock的方法（事务的配置方式见第3点）
     
3. 继承`com.yonyou.uap.billcode.BillCodeGeneratorNeedAddTransaction`,覆盖此类中唯一的public方法getBillCode方法，直接调用父类的方法，添加上`Requires_new`事务，特别的，对spring注解事务：  

    	public class BillCodeGeneratorWithTransaction extends
    			BillCodeGeneratorNeedAddTransaction {
    		@Transactional(propagation = Propagation.REQUIRES_NEW)
    		public String[] getBillCode(BillCodeEngine engine,
    				IBillCodeEngineLock lock, int num, boolean isInsertPrecode)
    				throws BillCodeException {
    			return super.getBillCode(engine, lock, num, isInsertPrecode);
    		}
    	}
	
	
4.继承编码规则的抽象类`com.yonyou.uap.billcode.AbstractBillCodeProvider`，实现三个抽象方法，三个抽象方法获取的对象分别为1，2，3中类的实例。
     
5.调用4中实现类的方法,传入参数，生成业务对象编号	

#### 二、对于业务对象编号的编码规则含有业务对象信息编码元素的情况

1. 和没有业务对象信息编码元素的情况一样(同1)
2. 和没有业务对象信息编码元素的情况一样(同2)
3. 和没有业务对象信息编码元素的情况一样(同3)
4. 和没有业务对象信息编码元素的情况一样(同4)   
5. 实现从业务对象中取值的接口`com.yonyou.uap.billcode.model.IBillVOFieldValueFetcher`，可以是对所有业务对象通用的，也可是针对某一个业务对象的取值方法
6. 将业务对象和5的实现类封装入`com.yonyou.uap.billcode.model.BillCodeBillVO`
7. 传入参数，生成业务对象编号

### 实现编码规则组件的自定义替换

#### 一、自定义编码元素处理器
1. 复制本jar包中`com/yonyou/uap/billcode/BillCodeEngineContext.xml`到任意'类路径A'
2. 实现编码元素处理器接口：`com.yonyou.uap.billcode.elemproc.itf.IElemProcessor`
3. 在编码规则引擎上下文配置文件（步骤1中复制的`BillCodeEngineContext.xml`）中'添加'或'替换'元素处理器

#### 二、自定义序列号编码元素的序列号生成方式
1. 同上步骤1
2. 实现编码元素处理器接口：`com.yonyou.uap.billcode.sngenerator.ISNGenerator`
3. 在编码规则引擎上下文配置文件（步骤1中复制的`BillCodeEngineContext.xml`）中'添加'或'替换'序列号生成器

#### 三、自定义系统时间编码元素的时间获取方式
1. 同上步骤1
2. 实现编码元素处理器接口：`com.yonyou.uap.billcode.sysdate.ISysDateProvider`
3. 在编码规则引擎上下文配置文件（步骤1中复制的`BillCodeEngineContext.xml`）中'替换'时间获取方式

#### 四、自定义唯一性随机编码生成类（不需要业务对象编号有业务意义时，此类提供唯一性的无业务意义的编码）
1. 同上步骤1
2. 实现编码元素处理器接口：`com.yonyou.uap.billcode.randomcode.IRandomCodeGenerator`
3. 在编码规则引擎上下文配置文件（步骤1中复制的`BillCodeEngineContext.xml`）中'替换'随机编码生成类

*注意：自定义编码规则组件的上下文，需要在程序启动时调用`com.yonyou.uap.billcode.BillCodeEngineContext.getInstance('A')`将引擎上下文A载入内存（否则将走默认的的上下文配置 `com/yonyou/uap/billcode/BillCodeEngineContext.xml`）*


### 默认的编码规则上下文文件内容：（参见jar包中 `com/yonyou/uap/billcode/BillCodeEngineContext.xml`）

>     <?xml version="1.0" encoding="UTF-8"?>
>     
>     <billCodeEngineContext>
>     	<!--如果对编码没有具体的要求，可以获取随机码，此处是随机码生成器-->
>     	<randomcode>com.yonyou.uap.billcode.randomcode.imp.RandomCodeGenerator</randomcode>
>     	<!--编码规则元素中系统时间字段类型，获取系统时间用到的类 -->
>     	<sysdate>com.yonyou.uap.billcode.sysdate.imp.DateProviderJavaDate</sysdate>
>     	<!--编码规则元素中流水号字段，流水号相关获取类 -->
>     	<sngenerators>
>     		<generator>com.yonyou.uap.billcode.sngenerator.imp.PureDigitalSNGenerator
>     		</generator>
>     		<generator>com.yonyou.uap.billcode.sngenerator.imp.LetterDigitalSNGenerator
>     		</generator>
>     	</sngenerators>
>     	<!--编码规则元素处理引擎 -->
>     	<elemprocessors>
>     		<processor>com.yonyou.uap.billcode.elemproc.imp.ConstElemProcessor
>     		</processor>
>     		<processor>com.yonyou.uap.billcode.elemproc.imp.SerialNumElemProcessor
>     		</processor>
>     		<processor>com.yonyou.uap.billcode.elemproc.imp.SysTimeElemProcessor
>     		</processor>
>     		<processor>com.yonyou.uap.billcode.elemproc.imp.RandomStrElemProcessor</processor>
>     	</elemprocessors>
>     </billCodeEngineContext>
