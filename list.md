#二手房列表页迁移详细设计
###route 配置
```php
<?php
    $config['mappings'][$prefix.'List_ListController'] = array(
        '^/[a-z]+/sale/?(\?.*)*', //匹配无搜索和筛选参数的列表页如 /sh/sale/?a=b
        '^/[a-z]+/sale/[a-z-]+/?(\?.*)*', //匹配只有一个区域+板块搜索参数的列表页 如 /sh/sale/pudong-qingpu/?a=123
        '^/[a-z]+/sale/[a-z-]+/a\d+_\d+-b\d-(x|\d+_\d+){1}-(x|\d+_\d+){1}-\d.*' //匹配严格类型的列表页 搜索筛选URL 如 /sh/sale/all/a500000_1000000-b0-0_50-x-3?a=123
    );
?>
```

###To Do List
    1. LB修改
    2. 确认是否可放开所有 sh/sale 类型的URL
    
###列表页类型
    1. 搜索列表页
    2. 筛选列表页
    3. 同小区房源列表页
    4. 学区房列表页
    5. 附近房源列表页
    6. 同首付房源列表页

### jock js 模块
    1. page/dom.dom/ui.rSearchpage/event/ui.autocomplete/ui.exposure/ajax/utils.base/
    
###头部
    1. topbar c  
        * 模块
            * LOGO (url index)
            * current city (url city list)
            * home logo （url index）
                * 交互 `如果是首页则不显示该图标`
            * user logo 显示个人消息数量（数据来自问答）
                * 交互 未登录灰色；登录后绿色，并显示被回答个数圆圈
                * question `需要个人消息数量接口`
            * favorite logo
                * 交互 无收藏灰色，有收藏变绿，并显示收藏个数圆圈
                * question `需要收藏数量接口`
        * 出现页面
            * 搜索列表页
            * 筛选列表页
            * 学区列表页
            * 附近房源列表页
        * 不出现页面
            * 同小区房源列表页 （该页面显示 灰色文字区域）
        * 文件位置
            * component/user/touch/common/topbar/TopBar.phtml
    2. 列表导航
        * 模块
            * 二手房
            * 新房
            * 租房
            * 商业地产
            * 问答
        * 出现页面
            * 筛选列表页
            * 搜索列表页
        * 文件位置
            * component/user/touch/topbar/NavBar.phtml
    3. 灰色文字头
        * 模块
            * 返回
            * 同小区页 显示 小区名 
            * 学区页 显示 学区名+学区房 
            * 附近房源页 显示 [区域名]+附近房源
        * 出现页面
            * 同小区房源页
            * 学区房列表页
            * 附近房源页
        * 文件位置
            * component/user/touch/common/header/ListHead.phtml
    
    4. 头部伪代码实现 （3个component 按需调用）
        * 根据当前所处的页面 判断该让哪个头的部分出现。即
        ```php
        <?php
            if ($topbar == 1) {
                //topbar  该值在 controller 控制
                $this->component('User_Touch_Common_Topbar_TopBar', array(
                    //内部再控制该显示出现具体的元素
                ));
            }
            if ($navbar == 1) {
                //navbar  该值在 controller 控制
                $this->component('User_Touch_Common_Topbar_NavBar', array(
                ));
            }
            if ($listHeader == 1) {
                //listHeader  该值在 controller 控制
                $this->component('User_Touch_Common_Head_ListHead', array(
                ));
            }
        ?>
        ```
            
###筛选项
    * 筛选页、搜索列表页  默认出现 （1，2，3项 4，5，6折叠隐藏  7：首页进入时置顶显示 否则折叠）
        1. 区域
            1.1 板块 (只有选择区域时才会显示)
        2. 价格
        3. 户型
        4. 面积
        5. 房龄
        6. 类型
        7. 搜索框 （首页搜索进入时会置顶 from=new_search 在最后）
    * 学区房、同小区房源、附近房源  列表页 （展示1，2 折叠3，4，5，6项  不含区域条件）
        1. 价格
            1.1 板块 (只有选择区域时才会显示)
        2. 户型
        3. 面积
        4. 房龄
        5. 类型
        6. 搜索框
    * 需要筛选项接口
    * 筛选项URL规则
        * domain/city/区域-板块/价格-户型-面积-房龄-类型 (即a0_0-b0-x-x-0 条件都不限)
            * 区域 all (不限) || 区域（板块不限）|| 区域-板块
            * 价格 a0_0 (不限) || a0_500000 (50W以下) || a500000_100000 (50-100W)
            * 户型 b0 (不限) || b1(一室) || b2 (二室)
            * 面积 x (不限) || 0_0 (不限) || 0_50 (50平米以下) || 50_100 (50-70平米)
            * 房龄 x (不限) || 0_2 (2年以内) || 2_5 (2-5年)
            * 类型 0 (不限) || 1 (公寓) || 2 (别墅) 
    * 筛选后请求方式
        * AJAX 方式请求 局部刷新列表 
    * 注意点
        * 局部刷新需要同时按规则 显示/隐藏 筛选项
    * 文件位置
        * component/user/touch/common/list/ListFilter.phtml

###列表
    * 模块 （搜索命中小区时 显示 1，2项 并且区域筛选荐不显示 否则只显示 第2项 ）
        1. 小区介绍 （小区名称、均价、涨跌状态 进入单页链接）
        2. 列表 (非AJAX请求时 展示数据整个页面（包含头）  AJAX请求时 只显示列表内容(往下滑动翻页时用))
        3. 分页功能 (ajax append) 
    * 文件位置 (component)
        1. component/user/touch/common/list/List.phtml
    

###搜索零结果推荐 Ajax

###筛选零结果推荐 Ajax

###新房推荐 Ajax    

###底部APP广告

###所需接口
    1. 个人消息数量接口
    2. 收藏数量接口
    3. 筛选项接口
    4. 房源列表搜索接口
    5. 搜索零结果推荐接口
    6. 筛选零结果推荐接口
    7. 新房推荐接口
###Question
    1. 同首付房源功能如何做？PRD里面没有

###伪代码设计
    * 文件说明及位置
        1.controller controller/user/touch/anjuke/list/
