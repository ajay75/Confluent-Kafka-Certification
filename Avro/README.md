 **Avro**
**Primitive Types**  
```
    1. null   
    2. boolean  
    3. int (32 bit)  
    4. long (64 bit)    
    5. float (32 bit)  
    6. double (64 bit)  
    7. byte[] (8 bit)  
    8. string (char sequence)  
```
**Complex Types**  
```
    1. record  
    2. enum  -- type symbols  
    3. array -- type items   
    4. map   -- type map  
    5. union -- same as array    
    6. fixed  -- type fixed   
```    
**Avro Schema Definition**  
```
namespace (required)
type (required) => record, enum, array, map, union, fixed
name (required)
doc (optional)
aliases (optional)
fields (required) {
    name (required)
    type (required)
    doc (optional)
    default (optional)
    order (optional)
    aliases (optional)
```

**Schema Registry stores**  
 ```All schemas in a Kafka topic _schemas defined by kafkastore.config = _schemas (default) which is a single partition topic with log compacted.```