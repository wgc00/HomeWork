###一、思路

  - 创建表内容：
       
       
       
    1.我们要创建三个表分别是：province（省份表）、
    city（城市表）、county（县/区表）
    
    2.分别在三个表上添加字段分别是：id（编号）、name（名称）、number（所属的编号）
    
    3.在每个表的number字段上添加外键（references）和省份表与城市表关联；城市表与县/区表关联
    注意：创建表时每个字段名最后不要一样，否则，多表查询时避免冲突。
    
  
  
  - 查询表的内容：
  
     
     1.首先我们先俩表进行内关联查询，是用inner join  和 on 关键字进行查询（城市表）number 相等于（省份表）id关联
     条件是查询出那个省份的有哪些城市。
    
     例子：
            select * from province p inner join city c 
            on p.priovinceId = c.cityNumber 
            where p.priovinceName = 'XX省';
              
     结果：查询出来的数据是XX省中有哪些城市。
     
     2.在1成立的条件上我们又要进行两表内关联查询，又用inner join 和 on 关键字进行查询 （城市表）id 相等于（县/区表）
     条件是查询出那个城市的有哪些县/区。
     
     例子：
            
           select * from city c inner join county c1 
           on c.id = c1.countyNumber 
           where c.cityName = "XX市";    
           
     结果：查询出来数据是XX市中有哪些县/区。
      
     3.当我们都能把1和2都理解时，就是进行三表查询了，在多表查询时我们要注意一点就是
     某个省份中他包含了有哪些城市，而城市中又包含了哪些县/区，1和2列子都为我
     们说明了表之间的关系。把 1 和 2 合并，就可以得到，查询某个省份就可以查询出省份中
     包含的城市，城市又包县/区。
      
      例子：
            
            select * from province p  inner join city c 
            on p.provinceId = c.cityNumber  inner join
            county c1 on c.cityId = c1.countyNumber 
            where p.provinceName = "XX省";
      
      结果：查询出了XX省中多个XX市，XX市又查询出多XX县/区。
         
        
        
     
    
     
## 二、union 与 union all 的区别？

      union:对两个结果集进行并集操作，不包括重复行，同时进行默认规则的排序

      union all: 对两个结果集进行并集操作，包括重复行，不进行排序；
      
      union all : 查询出来数量 大于或等于 union 查询出来的数量 
      
      注意：使用联合查询时，第一表字段要和第二个表的字段相等，不然会报错。
