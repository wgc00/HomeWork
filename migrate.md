##一、分表的总结

   1 .首先，我们要理解分表有有什么作用，在分表的过程有那么字段是必须分离出去的。
   
   - 作用：当数据库的数据量非常庞大时，要在数据库中执行sql语句是非常消耗时间和性能，为了
   减轻数据的负担，这时就要进行分表，来提高数据库的效率。
     
   - 对于字段我们必须满足数据库三大范式，如果不满足数据库三大范式，我们就应该把此字段分离处理。
  
---
   
   2 .通过下面的例子我们讲解如何分表：
    
   ①、我们要有两表一个是求职表、一个是全国地区表。
   
   求职表：
   
               create table  lagou_position(
                       pid	int(11),                              --职位id
                       city	varchar(100),                         --城市
                       district	varchar(100),                 --县/区
                       position	varchar(100),                 --职位
                       field	varchar(40),                  --发展方向
                       salary_min	double,                   --最低工资
                       salary_max	double,                   --最高工资
                       workyear	varchar(20),            
                       education	varchar(20),              --学历
                       ptype	varchar(100),
                       pnature	varchar(100),
                       advantage	varchar(100),             --待遇
                       published_at	datetime,
                       updated_at	datetime,
                       company_id	int(11),                 --公司id
                       company_short_name	varchar(30),
                       company_full_name	varchar(100),
                       company_size	varchar(100),           --公司招聘人数
                       financestage	varchar(100)
               );
    
    
   全国地区表：下载 [https://github.com/wgc00/HomeWork.git]
        
        DROP TABLE IF EXISTS `s_provinces`;
        CREATE TABLE `s_provinces` (
          `id` int(11) NOT NULL,
          `cityName` varchar(30) NOT NULL,
          `parentId` int(11) NOT NULL,
          `shortName` varchar(30) NOT NULL,
          `depth` int(1) NOT NULL,
          `cityCode` varchar(4) NOT NULL,
          `zipCode` varchar(6) NOT NULL,
          `mergerName` varchar(50) NOT NULL,
          `longitude` varchar(16) NOT NULL,
          `latitude` varchar(16) NOT NULL,
          `pinyin` varchar(30) NOT NULL,
          `isUse` int(1) unsigned zerofill DEFAULT NULL
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
   
                
    
   通过上面的表让我们了解到什么，表中没有满足第一范式的设计。
   
   
   
   ②、这时我们可以大概分为三个表，地区表（lagou_city）、职位表（lagou_positions）、公司表（lagou_company）
   
   ③、分完了三个表后，我就要理解，这些表要有哪些字段，还要满足第二范式依赖于主键；建立相关的依赖性。
   
   ④、地区表：要有城市id、省份、市、县、依赖于职位表。（地区表从全国地区表中截取）
      职位表：要有职位id、职位名称、发展的方向、最低工资、最高工资等。
      公司表：要有公司id、公司名称、招聘人数、公司标准等。
      
   
##构建思路，拼接sql语句。
     
   1 . 获取公司表，首先我们要明白公司要哪些字段：
   
   - 通过查询把所要的数据查询出来
    
          select  company_id, 
                  company_short_name, 
                  company_full_name, 
                  company_size, 
                  financestage
           from lagou_position;
   
   - 查询得出结果后创建一个公司表
   
            create table lagou_company as
            select  company_id, 
                              company_short_name, 
                              company_full_name, 
                              company_size, 
                              financestage
                       from lagou_position;
                       
                       
   2 .获取地区表，注意此此表中从全国表中获取，要理解一些递归的概念
           
   - 查询全省表：depth字段：0是国家、1是省、2是市、3是县/区
        
          select id, cityName as preovineName, parentId
          from s_provinces
          where depth = 1;  
          
          -- 创建一个省表：依赖于国家表，这里不写了
            
                create table  preovine_tem 
                 select id, 
                        cityName as preovineName, 
                        parentId
                 from s_provinces
                 where depth = 1;    
                  
          -- 创建一个市表：依赖于省表
                
                create table city_tem 
                 select id, 
                    cityName, 
                    parentId
                    from s_provinces
                    where depth = 2;
                    
          -- 创建一个县/区表 ：依赖于市表    
                 
                 create table county_tem 
                 select id, 
                 cityName, 
                 parentId
                 from s_provinces
                 where depth = 2;
                 
          -- 多表查询，创建地区表
          
                create table lagou_city as
                select ct.id,preovineName,cityName,countyName
                from provinces_tem p
                       inner join city_tem c on p.id = c.parentId
                       inner join county_tem ct on c.id = ct.parentId;
                
   
   在lagou_position表中添加一个cityId
   
        alter table lagou_position add column countyId int;
    
   在使用lagou_position表中city，district进行条件查询，并把cityId修改
   
       select * from lagou_position ls left join lagou_city lc
       on ls.city = lc.cityName and ls.district = lc.countyName ; 
       
       -- 修改
       
       set autocommit = 0;      -- 关闭默认提交
       update  lagou_position ls left join lagou_city lc
	   on  ls.city = lc.cityName and ls.district = lc.countyName 
	   set  ls.countyId  =  lc.id;
    
       commit ;                 --提交
                           
   
   3 .获取职位表，注意一点公司表依赖于此表
   
   - 首先要查询职位表要有哪些字段     
   
          select distinct pid,
                          cityId,
                          position,
                          field,
                          salary_min,
                          salary_max,
                          workyear,
                          education,
                          ptype,
                          pnature,
                          advantage,
                          published_at,
                          updated_at,
                          company_id
          from lagou_position
        
        
      
     
      
      
      
   
   
        
   
   
   
