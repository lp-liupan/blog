---
layout: post
title: swagger在laravel中的使用
date: 2019-07-19 11:34:01
categories: 开发小工具
---

# 开始前了解

>推荐一个比较完成的文档[https://learnku.com/laravel/t/7430/how-to-write-api-documents-based-on-swagger-php#b5a896](https://learnku.com/laravel/t/7430/how-to-write-api-documents-based-on-swagger-php#b5a896)  
<!-- more -->   
>swagger主要是用于给api接口写文档的，接口主要有几个关键点需要文档对其进行解释     

1. 请求方法
2. 请求的路径
3. 请求的参数
4. 返沪状态
5. 返回参数  

>另外强调一点，所有的swagger语句都要写在注释里面！

# 请求方法

>主要是指GET、POST等请求方法在swagger中应该如何表示。    
>如果一个模块或者说一个功能点需要多个接口，可以使用tags属性给接口打个标签。    

```php
/*
    //请求的方法写在开头，例如：GET、POST、DELETE等
    @SWG\GET(
        tags={"巡检"},                   //相当于给这个请求归类，一般按照模块划分，这里指这个请求属于巡检模块
        path="/checktaskitem",          //请求的路径
        summary="根据巡检员获取其巡检列表"  //对改请求的描述，主要说明其功能
    )
*/
```

{% asset_img swagger1.png %}  

# 请求参数

>参数主要有几种形式：

1. query string：	api/checktask?item_id=1
2. path: 	api/checktask/1
3. body:	post打包传值的形式    

## 简单一点的请求方式

>简单的请求方式就是指通过query或者path这两种形式进行传值的方式，因为不涉及到参数数组、对象的嵌套，因此有多少个参数就写多少个Parameter方法就可以。Parameter方法的参数配置参考：

```php
/*
    //请求的方法写在开头，例如：GET、POST、DELETE等
    @SWG\GET(
        tags={"巡检"},                   //相当于给这个请求归类，一般按照模块划分，这里指这个请求属于巡检模块
        path="/checktaskitem/failed",   //请求的路径
        summary="根据巡检id获取巡检违规项" //对改请求的描述，主要说明其功能
        
        //请求参数
        @SWG\Parameter(
            name="item_id",      //参数key
            description="巡检id",//参数注释
            required=true,      //参数是否必填，注意：这里填写boolean值，不能填写字符串
            type="integer",     //参数值类型，注意：数值为integer，字符串为string
            in="query",         //参数传递的形式，注意：这里主要有query、path、body（比较麻烦，单独讨论）
        )
    )
*/
```

{% asset_img swagger2.png %}   

## 复杂一点的请求方式

>使用post或者put等方法的时候往往涉及到了对象、数组的多层嵌套进行传值。在swagger中书写起来比较麻烦一点。对Parameter传递的参数中需要添加一个Schema方法，该方法作用就是可以在文档中生成一个model，展示请求参数的注释和数据结构。

>如果Schema传递的是个object，那么在Schema方法中的Property就是它的每个属性，如果Schema传递的是个数组，并且数组中每一项都是一个object，那么需要在Property方法中添加Items方法，然后在Items中继续使用Property。

```php
/*
     *  @SWG\PUT(
     *   path="/checktaskitem/{item_id}",       //注意：当有参数是以path的方式传递，要在这里加上{}
     *   tags={"巡检"},
     *   summary="提交巡检结果",
     *
             @SWG\Parameter(                    //这里就是使用path方式传递的参数
                name="item_id",
                required=true,
                type="integer",
                description="巡检id",
                in="path"
             ),
             @SWG\Parameter(                  //当有body方式传递时，单独用一个parameter，其类型定义为body
               name="person",
               in="body",
               required=true,
               
               @SWG\Schema(                   //Schema的作用时可以有一个model对参数进行解释，具体作用用截图展示
                   type="object"              //给传递的参数定义类型，目前是对象，也可以定义为array
                   
                   @SWG\Property(                      //注意：当使用了Schema后，每个字段需要用Property来定义
                        property="item_serialno",      //字段名
                        type="string",                 //字段类型
                        description="和对讲平台给的巡检单号", //字段的注释
                   ),
          
                   @SWG\Property(
                        property="item_resultdata",
                        type="array",                 //注意：这里把这个字段定义为了数组（也可以定义为object）
                        description="巡检结果具体项",
                        
                        @SWG\Items(                  //注意：如果父级是[]子级是{}，需要加入Items，如果父级{}则不用添加Items
                            @SWG\Property(
                               property="item_id",
                               type="integer",
                               description="巡检项id"
                            ),
                            @SWG\Property(
                                property="children",
                                type="array",         //同上，这里又有一个数组，并且使用了Items说明数组中嵌套了对象
                                description="具体巡检内容",
                                
                                @SWG\Items(
                                    @SWG\Property(
                                        property="item_id",
                                        type="integer",
                                        description="巡检项id"
                                    ),
                                )
                            ),
                        )
                   ),
               )
           ),
      )
     
*/
```
{% asset_img swagger3.png %}
{% asset_img swagger4.png %}

# 返回参数

>返回的参数中必定存在各种数组、对象的嵌套，我们在写swagger的时候其实可以参考复杂类型的参数传递方式，同样都是数组包裹对象，或者对象包裹数组，本质上是一样的。只是原本写在Parameter方法中的Schema现在要写在Response中。

```php
/*
 *  @SWG\Get(
 *   path="/checktaskitem/failed",
 *   tags={"巡检"},
 *   summary="根据巡检id获取该巡检违规项具体信息",
 *
 *  @SWG\Parameter(
 *      name="item_id",
 *      description="巡检id",
 *      required=true,
 *      type="integer",
 *      in="query"
 *  ),
 *
 *  @SWG\Response(                                             //定义响应参数的时候，首先使用Response
 *         response=200,                                       //响应
 *         description="基本信息",
 *         @SWG\Schema(                                        //同样的这里使用Schema表示可以在model中查看注释
 *            type="array",
 *                  @SWG\Items(                                //如果父级为[]子级为{}则要是用Items，其中包含了子级{}中的属性的类型和注释
 *                      @SWG\Property(
 *                         property="item_id",
 *                         type="integer",
 *                         description="巡检项ID"
 *                     ),
 *                     @SWG\Property(
 *                         property="inspection_id",
 *                         type="string",
 *                         description="巡检单id"
 *                     ),
 *                      @SWG\Property(
 *                         property="item_name",
 *                         type="integer",
 *                         description="巡检项名称"
 *                     ),
 *
 *                     @SWG\Property(
 *                         property="enterprise_name",
 *                         type="string",
 *                         description="企业名称"
 *                     ),
 *                  ),
 *
 *                ),
 *     ),
 *   @SWG\Response(response=401, description="用户验证失败"),
 *   @SWG\Response(response=500, description="服务器错误"),
 * )
 *
 */
```
{% asset_img swagger5.png %}
{% asset_img swagger6.png %}

# 了解内容

>很多时候我们的接口是需要带上一个token才能进行使用的，如果没有token就会锁住该接口，不允许访问。这时我们可以在最后加上security，如果没有写则表示这个接口不需要使用token，比如登录接口就不需要。

```php
/**
 *  @SWG\Get(
 *   path="/checktaskitem/failed",
 *   tags={"巡检"},
 *   summary="根据巡检id获取该巡检违规项具体信息",
 *  @SWG\Parameter(),
 *  @SWG\Response(),
*     security={
*          {
*              "Bearer":{}
*          }
*      },
* )
*/
```

{% asset_img swagger7.png %}
