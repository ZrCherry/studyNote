### MyBatis-Plus

Mybatis-plus是在Mybatis的基础上做了增强（封装好一些CRUD方法），简化开发，提高效率。

#### 核心功能

1. 代码生成器

   **AutoGenertor** 类是Mybatis-Plus的代码生成器，可以快速生成Entity，Mapper，MapperXMl，Service，Controller等模块代码。

2. CRUD接口

   CRUD接口是Mybatis-Plus的核心功能中的核心功能，主要提供了两个接口，再service层提供了IService接口，在Dao层提供了BaseMapper接口，这两个接口实现了一些通用的CRUD代码。在写接口时，只要在对应的层继承对应的接口即可实现dao层0编码。

3. 条件构造器

   虽然MP实现了许多基础的CRUD方法，但是我们往往需要使用一些自定义的操作，此时我们就可以使用条件构造器，而不是继续去回到Mybatis去写sql语句。自定义的sql语句也可以通过条件构造器来实现。

