
<!--more-->

# Json.Net 序列化实体为空

标签（空格分隔）： 爬坑记录

---

## 原因

在 Json.Net里, 如果有 Public 的属性不想被序列化/反序列化, 有两种方法 

1. 给属性加上`[JsonIgnore]` , 这个类序列化时会忽略这个属性
2. 在类上面加`[DataContract]` , 这个类里只有标记了[DataMember]的属性才会被序列化/反序列化

Json.Net 的文档链接: [http://www.newtonsoft.com/json/help/html/DataContractAndDataMember.htm][1]

## 解决方法

在序列化时使用自定义的`ContractSolver`

上代码

```csharp

 [Test]
 [Category("TestsTest")]
 public void NewtonsoftTest_DataMemberInDataContractClass_ReturnEmpty()
 {
     var json = JsonConvert.SerializeObject(new MyClass());

     TestContext.WriteLine(json);
     Assert.That(json, Is.EqualTo("{}"));
 }

 [Test]
 [Category("TestsTest")]
 public void NewtonsoftTest_DataMemberInDataContractClass_UsingCustomResolver_Return()
 {
     var json = JsonConvert.SerializeObject(new MyClass(), new JsonSerializerSettings
     {
         ContractResolver = new AllPropertiesResolver() //**here**
     });

     TestContext.WriteLine(json);
     Assert.That(json, Does.Not.Contain("{}"));
 }

 public class AllPropertiesResolver : DefaultContractResolver
 {
     protected override JsonProperty CreateProperty(MemberInfo member, MemberSerialization memberSerialization)
     {
         JsonProperty property = base.CreateProperty(member, memberSerialization);

         //property.HasMemberAttribute = true;
         property.Ignored = false;

         //property.ShouldSerialize = instance =>
         //{
         //    return true;
         //};

         return property;
     }
 }
 
 /// <summary>
 /// 测试 这个类被标记了[DataContract], 如果使用默认的序列化设置, 那么两个属性都不会输出出来
 /// 如果自定义了 AllPropertiesResolver, 则是忽略所有的Attribute, 输出应该正常了
 /// </summary>
 [DataContract]
 class MyClass
 {
     public string S { get; set; }

     public bool B { get; set; }
 }

```


[http://stackoverflow.com/questions/32424952/json-net-how-do-i-serialize-or-deserialize-all-property-even-include-non-data][2]

[http://stackoverflow.com/questions/11055225/configure-json-net-to-ignore-datacontract-datamember-attributes][3]


## 探讨

如果一个属性同时被标记为 `[DataMember]` 和 `[JsonPropertyAttribute]` 呢?

> Json.NET attributes take presidence over standard .NET serialization attributes, e.g. if both JsonPropertyAttribute and DataMemberAttribute are present on a property and both customize the name, the name from JsonPropertyAttribute will be used.

## Reference

* [http://james.newtonking.com/archive/2009/10/23/efficient-json-with-json-net-reducing-serialized-json-size][4]


  [1]: http://www.newtonsoft.com/json/help/html/DataContractAndDataMember.htm
  [2]: http://stackoverflow.com/questions/32424952/json-net-how-do-i-serialize-or-deserialize-all-property-even-include-non-data
  [3]: http://stackoverflow.com/questions/11055225/configure-json-net-to-ignore-datacontract-datamember-attributes
  [4]: http://james.newtonking.com/archive/2009/10/23/efficient-json-with-json-net-reducing-serialized-json-size
