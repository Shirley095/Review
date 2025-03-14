# Review
难点
1、动态评分模板的灵活性与扩展性
难点：评分项与权重需支持动态配置，且需适应不同实验项目的多样性。
解决方案：
设计可扩展的评分模板表结构：
sql
复制
CREATE TABLE score_templates (
    id INT PRIMARY KEY,
    project_id INT,
    category VARCHAR(50),
    weight DECIMAL(5,2),
    FOREIGN KEY (project_id) REFERENCES projects(id)
);

前端通过递归组件渲染动态表单，后端提供模板解析与总分计算服务。

2、前后端分离的协作与联调

难点：接口定义不一致、数据格式错误等问题频发。

解决方案： 	

使用Swagger生成API文档，确保前后端对齐。

制定统一的数据传输规范（如RESTful API + JSON）。
3、高并发场景下的实时数据同步

难点：选题状态、成绩发布等操作需实时更新，避免数据不一致。

解决方案：

使用WebSocket实现实时通信：

javascript
复制
const socket = new WebSocket('ws://your-server/notify');
socket.onmessage = (event) => {
    updateData(JSON.parse(event.data));
};
结合Redis缓存高频访问数据（如选题状态），降低数据库压力。


4 系统分析与设计（工科线上...曹）
4.1 系统需求分析
4.1.1 系统功能模块分析
系统在开发过程中将采用角色分块编写策略，根据用户模块管理需求，对各个功能进行不同界面的开发，下面将对系统的各个功能模块设计需求进行简要介绍：
（1）实验编制
实验编制部分主要完成对实验报告题干的编辑工作，输入题干信息，选择题目类型、设置参考答案、各题所占分值及重复度阈值。
（2）题库管理
题库管理部分对编制好的实验报告进行管理，可对报告进行删除、预览或重新编辑。
（3）教师管理
教师管理部分对教师信息及每位教师所教课程进行管理，可增加或删除教师信息并对每位教师所教课程进行绑定操作。
（4）班级管理
班级管理包括对班级的创建以及学生名单的上传，同时对学生信息进行管理。
（5）课程管理
课程管理部分可对课程进行添加，也可完成对每个课程涉及班级的添加以及每个课程涉及题库的添加工作。
（6）实验报告管理
实验报告部分分为学生端和教师端两个模块，学生端可供学生查看需要完成的实验报告并进行填写、提交以及后续的查分操作。教师端可查看学生完成的实验报告，也可进行各部分分数的修改。
4.1.2 系统用户功能分析
根据系统用户分为三种，为不同角色用户设置不同权限。如图 4.1 展示了教师、学生及管理员三种用户与其它用例之间的关系。

（1）管理员用户
管理员用户是管理系统中权限最大的，可以对除了教师所有的功能模块进行操作，并负责教师信息的管理和维护。
（2）教师用户
教师用户的主要功能需求包括对发布实验的项目评分项和评分比例的设置。且教师可以对实验编制、题库管理、班级管理、课程管理、实验报告管理模块的相应功能进行操作。
（3）学生用户
学生用户主要功能需求包括根据教师设置的项目评分项和评分比例，完成自我评分取长补短，并提交自己所学课程的实验资料，因此只需对个人所涉及对实验报告管理模块进行操作。

（旧）4.1.3 实验报告评分分析 
实验报告在线评分是整个系统的核心业务，在系统设计过程中一定要保证评分流 
程的顺利进行，确保题目类型与评分模型中的相应程序准确匹配，也要保证评分算法 
得出结果的准确率。

4.1.4 系统性能需求分析 （B/S）
（留）计算机技术及网络的快速发展，为该系统的开发提供了技术的可行性，各种成
熟的软硬件环境，为系统开发带来了极大便利。同时 Web 开发技术已经经历过长久
的考验，B/S架构通过Web服务器与数据库服务器进行数据交互，不仅使前期开发提供了方便，同时有利于后期的维护及更新。

在这些基础上，系统的性能也要跟上。考虑课程需求，系统使用用户较多，对系统稳定性
要求也就相对较高。根据系统获取、查询等响应时间需求，将系统最大响应时间设置为<10 秒。

（留）同时，由于系统储存了大量用户和课程信息，如果出现错误会影响课程的正常进行、影响学生成绩，所以系统的安全性也至关重要。因此要搭建一个全面的系统安全防范性措施，采用分级设计，不同级别的访问权限设置不同，保障用户信息的安全性。

最后，考虑到线上实验报告智能评价系统不管是在节省人力物力、还是提高教学效率上
都有良好的表现，在日后一定有较多的应用。在系统设计方面要保证业务逻辑的可靠
性，使系统操作流程无误，与此同时还需对系统设置二次审核功能，保障信息分析处
理的准确性


B/S
什么是B/S架构？
B/S架构，即浏览器/服务器架构，是一种网络架构模式，将系统功能实现的核心部分集中到服务器上，简化了系统的开发、维护和使用。客户端只需要安装一个浏览器，通过Web服务器与数据库服务器进行数据交互。B/S架构利用了Web浏览器技术和Internet协议，实现了异构系统的连接和信息的共享。

B/S架构的分层：
第一层表现层：主要完成用户和后台的交互及最终查询结果的输出功能。
第二层逻辑层：主要是利用服务器完成客户端的应用逻辑功能。
第三层数据层：主要是接受客户端请求后独立进行各种运算。


4.2 系统功能整体设计
4.2.1 系统架构设计
MVC模式和三层架构，是分别从两个不同的角度去设计的，但目的都是“解耦，分层，代码复用等”。

（根据设计）系统整体采用前后端分离架构，前端应用 Vue.js 框架进行搭建，其中的 HTML、CSS 以及 JS 语言对页面的结构、样式及功能部分进行编写。后端应用 ASP.NET Core，使用SpringBoot语言，实现对数据业务逻辑的管理及数据库的映射。

算法部分采用单独的阿里云函数计算进行部署，前端及阿里云函数计算 FC 均通过 WebAPI 进行访问后端主程序，后端主程序基于 MVC 架构，但由于前后端分离，前端负责 View 开发，后端负责开发 Model 及 Controller，Model 中的实体类的属性与数据库中的字段值保持一致，编写好的 Model 通过 EF Core 将实体映射到数据库表，Controller 则负责具体模块的业务流程控制，Controller 处理前端控制器发送过来的请求并返回结果视图。系统整体架构图如图 4.2 所示：


4.2.2 画图

4.2.3 数据库设计
从以上 7 个信息表中可以看出课程信息表的班级、实验和班级信息表及实验信息表中的数据是相对应的，这种对应关系还有很多，所以各个表之间是相互关联、相互依赖的。

（旧）4.3 本章小结 
本章通过分模块、分角色的方式对系统功能进行了需求分析，并对评分的准确性 
及系统的性能ᨀ出要求。针对以上系统需求，对系统的整体架构、各模块功能及数据 
结构进行了相应设计。

