---
title: RFC 7234
date: 2017-10-23
---
## 1 导言
  HTTP通常用于分配信息的系统，在这里性能可以通过利用响应存储得到提升，这份文档定义 HTTP/1.1关于缓存和再利用响应消息的部分.  

  一个HTTP缓存除了包括响应消息的本地存储控制还包括检索，删除，控制存储消息本身.一个缓存了可缓存的响应和等效的普通响应相比使用更少的响应时间和带宽消耗.任何的用户(client)和服务(server)都可以使用缓存，但是被当做隧道(tunnel)的服务不能使用.  

  共享缓存(shared cache)的响应存储可以被多用户再次利用，通常共享缓存被部署为中介的一部分.相反的私有缓存(private cache)只能被一个用户检查到，通常私有缓存被部署为用户代理的一部分.  

  在HTTP/1.1中缓存通过再利用先前的响应满足现有请求而达到明显提高效率的目标.一个已存储的缓存被认为"新鲜"(fresh)必须先满足4.2定义部分，那么这个响应可以再利用并且不用"检验"(validation：向源服务器检查看缓存响应是否对于这个请求仍然有效).一个新鲜的响应可以每次重利用因此可以有效的减少延迟的网络管理.当缓存响应不再新鲜是，它也许还可以利用比如通过检验再更新新鲜度(4.3)或者服务源unavailable(4.2.4).  
  <!--more-->

### 1.1 一致性和错误处理
  关键字如"必须"，"必须不"，"必要的"，"将会"，"将不会","应该"，"应该不"，"建议"，"也许"，"选择性的"这些在文档中出现的都将在[RFC2119]得到解释.    

  一致性准则和常规错误处理都在[RFC7230]中得到定义.

## 2 缓存系统概览
  正确的缓存系统应该保持HTTP传输中的语法，同时应该消除已经存在于缓存中的传输信息.虽然缓存行为在HTTP中完全是一个可选项.但是你可以将再利用缓存响应当做可取的并且这种再利用是不需要要求或者本地配置来强制的默认行为.因此HTTP的缓存配置是用来强制性规定缓存行为来满足一些我们想要的特性，而不是像托管一样总是缓存再利用一些特定的响应.    

  每个缓存记录包含一个缓存键和一个或者多个http响应，这些响应对应着先前使用同一个键值的请求.一个最常见形式的缓存记录为成功的检索请求： ie,200(OK).然而，它也可能缓存一个永久性重定向301，否认结果404，不完全结果206，或者缓存一个除了GET以外的方法.  

  基本的缓存键值包含请求方法和目标URI，因为HTTP缓存通常只缓存GET方法返回的请求，许多缓存简单的去掉其他方法并只用URI当做基本的缓存键值.  

  如果一个请求的目标为内容协商，它的缓存体也许会包含多种响应，每种不同响应通过第二键值做区分(4.1)

 ## 3 在缓存中存储响应的过程
  缓存前提条件：
  * 请求方法必须能被缓存所理解并且被定义为可缓存类型
  * 响应状态码必须能被缓存理解
  * 缓存指令"no-store"(5.2)不能出现在请求或者响应头中
  * 共享缓存在响应中缓存指令不能出现"private"(5.2.2.6)否则共享缓存不生效
  * 共享缓存中权限认证头信息(4.2 of[RFC7235])不能出现响应头，除非响应中有明确允许其可以缓存(3.2)
  * 其中之一的响应：
      * 包含一个Expires头信息(5.3)
      * 包含一个max-age 的缓存指令(5.2.2.8)
      * 在共享缓存缓存中，包含一个 s-maxage缓存指令(5.2.2.9)
      * 包含一个Cache Control 扩展(5.2.3)允许其可以被缓存
      * 包含一个被定义为可缓存的状态码(4.2.2)
      * 包含一个public的缓存指令(5.2.2.5)  
  注意以上的满足条件都可以被cache-control扩展所重写(5.2.3)    

  在这段内容中，如果表现出与缓存相关的行为时，这时缓存已经"理解"一个请求方法或响应状态码.  

  注意，通常系统中，一些缓存将不会存储一个既没有明确的过期时间也没有缓存验证器的响应.但是缓存也没禁止缓存这样的响应.  

  ### 3.1 存储不完全响应
  ### 3.2 存储权限认证请求的响应
    除非缓存指令指定，否则共享缓存不能用于缓存拥有权限认证请求返回的响应.(4.2 of [RFC7235]).
    在这份规格中，下面的Cache-Control的响应指令中有这些指令： must-revalidate， public， s-maxage.  

    注意在缓存响应中包含"must-revalidate" 和/或"s-maxage"的响应指令则不允许过期存储被共享存储(4.2.4). 特别是在响应中要么包含"max-age=0, must-revalidate"或者"s-maxage=0"则表明后续请求都必须通过源服务器否认验证才能使用.
  ### 3.3 混合部分内容

  ## 4 从缓存到响应的构成
   当发起请求时，利用缓存必须满足以下条件：
   * 请求为有效请求URI，并且在缓存中能匹配到
   * 请求方法和所匹配的缓存允许其用来展现请求
   * 所选择的请求头被匹配的缓存所命中(nomiated：4.1)
   * 请求头中不能包含pragma = no-cache的请求头(5.4),或者请求头中有缓存指令包含no-cache,除非存储的缓存已经成功的验证.
   * 存储的响应没有包含no-cache缓存指令，除非已经验证成功
   * 存储的响应时一下之一： 
        * 新鲜(fresh:4.2)
        * 允许其过期响应(4.2.4)
        * 验证成功(4.3)
    
    注意上面的条件都有可能被cache-contral扩展所重写(4.2.3)
    当一个存储好的响应被用来满足一个没有验证的请求时，这个缓存必须产生一个Age头(5.1),并且替换在响应中任何等价于current_age的值(4.2.3)

   ### 4.1 使用Vary当做第二存储键值
   ### 4.2 新鲜度
   一个新鲜响应代表着它的生命周期还没过期，相反的是过期响应.    

   一个响应的新鲜度生命周期是从其源服务器产生到过期的时间，明确过期时间：当存储的响应过了期明确的过期时间这就意味着响应必须通过源服务器验证后才能使用。当一个缓存没有明确的过期时间时启发式过期时间就会被缓存赋予.  

   一个响应的年龄(age)表示其从源服务器所产生到当前的时间差，或者从源服务器验证成功到目前的时间差.  

   当一个响应在缓存中新鲜时，它可以满足其后的相同请求并不需要访问源服务器，因此相当高效.  

   决定新鲜度的最主要机制通常为源服务器提供一个明确在未来过期的时间(5.3)，或者响应提供一个max-age指令(5.2.2.8).

   因为源服务器不总是提供一个明确的过期时间，缓存也可以允许其使用启发式过期时间(4.2.4)  

   具体的计算规则：
   ```
    response_is_fresh = (freshness_lifetime > current_age)
   ```
   freshness_lifetime定义在4.2.1;current_age定义在4.2.3  

   客户端可以在请求中使用max-age或者min-fresh的响应指令来放松或加固信所对应响应新鲜度的计算
(5.2.1)   

  注意新鲜度只能应用与缓存系统;它不能用户强迫客户端刷新其展示或者重新加载资源，第6部分将会解释缓存和历史机制的区别.  

  #### 4.2.1 计算新鲜度生命周期
  一个缓存可以通过如下最先匹配的规则来计算器生命周期(称:freshness_lifetime):
  * 如果缓存被共享并且响应中有s-maxage缓存指令
  * 如果响应中有缓存指令max-age
  * 如果头信息中存在Exoires，用它的值减去响应头的Date值
  * 使用启发式生命周期(4.2.2)
  ```
  //下面是 http权威指南 服务端新鲜度计算伪代码比较易懂
  sub server_freshness_limit
  {
    local($heuristic,$server_freshness_limit,$time_since_last_modify);
    $heuristic = 0;
    if ($Max_Age_value_set)
    {
      $server_freshness_limit = $Max_Age_value;
    }
    elseif ($Expires_value_set)
    {
      $server_freshness_limit = $Expires_value - $Date_value;
    }
    elseif ($Last_Modified_value_set)
    {
      $time_since_last_modify = max(0, $Date_value -
    　　　　　　　　 $Last_Modified_value);
      $server_freshness_limit = int($time_since_last_modify *
    　　　　　　　　 $lm_factor);
      $heuristic = 1;
    }
    else
    {
      $server_freshness_limit = $default_cache_min_lifetime;
      $heuristic = 1;
    }
    if ($heuristic)
    {
      if ($server_freshness_limit > $default_cache_max_lifetime)
      { $server_freshness_limit = $default_cache_max_lifetime; }
      if ($server_freshness_limit < $default_cache_min_lifetime)
      { $server_freshness_limit = $default_cache_min_lifetime; }
    }
    return($server_freshness_limit);
  }
  ```
  ```
  sub client_modified_freshness_limit
  {
    $age_limit = server_freshness_limit( ); ## From Example 7-2
    if ($Max_Stale_value_set)
    {
      if ($Max_Stale_value == $INT_MAX)
      { $age_limit = $INT_MAX; }
      else
      { $age_limit = server_freshness_limit( ) + $Max_Stale_value; }
      }
    if ($Min_Fresh_value_set)
    {
      $age_limit = min($age_limit, server_freshness_limit( ) -
    　　　　　　　$Min_Fresh_value_set);
    }
    if ($Max_Age_value_set)
    {
      $age_limit = min($age_limit, $Max_Age_value);
    }
  }
  ```

  注意计算过程并不会时钟抖动，因为所有的信息都来自源服务器.  

  当响应头有多余一个值的指令信息(e.g. 两个过期响应头，多个Cache-Contorl: max-age指令)，这些值都会被当做无效值.缓存鼓励其响应拥有无效生命新鲜度来让它过期.  

  #### 4.2.2 启发式(Heuristic)新鲜度计算
   如果源服务器没有提供一个明确的过期时间，缓存也许可以使用一定算法来利用其他头信息来(如Last-Modified)来推算一个合理的过期时间，这份文档并不会提供具体算法，但是可以为一些边界情况加一些限制。  

   缓存只有在没有一个明确的过期时间时才可以使用启发式新鲜度时间，根据在第3部分的缓存的前置条件。只有两种可能，一是响应状态码位可缓存的(6.1 of[RFC7231])且没有明确过期时间，二是被标记为可以缓存(e.g. 有"public"缓存指令)且没有明确过期时间.  

   如果存储响应有头信息Last-Modified(2.2 of [RFC7232]),缓存鼓励这个启发式过期时间为从现在到Last-Modified值之间差的某一比例.通常这个比例也许为10%.  
  ```
    freshness_lefetime = (date - Last-Modified)*10% 
  ```

   当启发式过期时间用于新鲜度生命周期，其中current_age的值已经超过24小时时缓存应该产生一个113 warn-code的头信息(5.5.4).  

   #### 4.2.3 计算年龄(age)
   头信息的Age通常用来表示从缓存获取响应的年龄时间.age的值通常由响应被产生时或被重新验证过起来估算.本质上来说Age的值由两部分之和，一是响应在缓存中驻存时间，二是响应在网络中转换的时间.   

   如下值用来age值的计算:  

   * "age_value"这个条目指的是头信息Age的值(5.1),这个值是以一种合适的算法系统计算而出，或者当它不存在的时候则为0

   * date_value这个条目的值为头信息Date的值  

   * now这个条目值指的是执行计算时的时间点  

   * request_time这个时间指的是请求从缓存中返回的时间点

   * response_time主机接受响应的时间点

   响应age可以通过两种完全独立的方法计算  

   * 1."apprent_age":如果在本地时钟和源服务器时钟合理同步时 结果=response_time - date_value. 如果解构等于负数，结果被零替换.  

   * 2."corrected_age_value",如果所有的缓存都遵循HTTP/1.1,缓存必须就必须将这个值理为与请求初始化成功相关，而不是与响应接受相关  
   ```
   apparent_age = max(0, response_time - date_value);
   response_delay = response_time = request_time;
   corrected_age_value = age_value + response_delay
   ```
  
  这两者相结合就是  

  ```
  corrent_initial_age = max(apparent_age, corrented_age_value);
  ```

  current_age 的值：
  ```
  resident_time = now -response_time;
  current_age  = corrented_initial_age + resident_time
  ```

  ```
  //这一版是http权威指南上的感觉更好理解
  $apparent_age = max(0, $time_got_response - $Date_header_value);
  $corrected_apparent_age = max($apparent_age, $Age_header_value);
  $response_delay_estimate = ($time_got_response - $time_issued_request);
  $age_when_document_arrived_at_our_cache =
  $corrected_apparent_age + $response_delay_estimate;
  $how_long_copy_has_been_in_our_cache = $current_time - $time_got_response;
  $age = $age_when_document_arrived_at_our_cache +
  $how_long_copy_has_been_in_our_cache;
  ```

  #### 4.2.4 提供过期响应
  一个"过期(stale)"响应要么有明确的过期信息或许是已经过期的通过启发式新鲜度计算的值.  

  如果通过缓存指令致使明确过期的响应(e.g, no-store, no-cache,must-revalidate,s-maxage, proxy-revalidate)，缓存一定不能返回过期响应.  

  除非失去连接一般缓存不会返回过期响应(i.e:不能连接到源服务器或者发现一个前进路径),另外明确指出可以返回过期响应也可以(e.g: 通过max-stale 请求指令)  

  缓存在返回过期响应时响应头应该有110 warn-code(5.5.1),同样的在失去连接情况下返回过期响应，响应头有112 warn-code  

  在前往一个没有Age头信息的响应如果响应已经过期缓存不应该产生一个新的warning头信息. 缓存不需要为一个仅仅因为传输而过期的响应而验证.

  ### 4.3 
  当一个缓存拥有一个或者多个缓存响应对应一个请求URI但是却无法提供给这个请求(e.g:因为这些缓存不新鲜，或者没有被选择4.1)，这样的话可以利用条件请求机制(conditional request machanism[RFC7232])转发请求给下一个到达的服务器来选择正确的响应来使用，一般可以通过更新存储的元数据或者更新一个完全新的响应.这个过程被称为验证存储响应.  

  #### 4.3.1 发送一个验证请求
  #### 5 Cache-Control
  ```
  //这份图表来自 RFC2616
   Cache-Control   = "Cache-Control" ":" 1#cache-directive
    cache-directive = cache-request-directive
         | cache-response-directive
    cache-request-directive =
           "no-cache"                          ; Section 14.9.1
         | "no-store"                          ; Section 14.9.2
         | "max-age" "=" delta-seconds         ; Section 14.9.3, 14.9.4
         | "max-stale" [ "=" delta-seconds ]   ; Section 14.9.3
         | "min-fresh" "=" delta-seconds       ; Section 14.9.3
         | "no-transform"                      ; Section 14.9.5
         | "only-if-cached"                    ; Section 14.9.4
         | cache-extension                     ; Section 14.9.6
     cache-response-directive =
           "public"                               ; Section 14.9.1
         | "private" [ "=" <"> 1#field-name <"> ] ; Section 14.9.1
         | "no-cache" [ "=" <"> 1#field-name <"> ]; Section 14.9.1
         | "no-store"                             ; Section 14.9.2
         | "no-transform"                         ; Section 14.9.5
         | "must-revalidate"                      ; Section 14.9.4
         | "proxy-revalidate"                     ; Section 14.9.4
         | "max-age" "=" delta-seconds            ; Section 14.9.3
         | "s-maxage" "=" delta-seconds           ; Section 14.9.3
         | cache-extension                        ; Section 14.9.6
    cache-extension = token [ "=" ( token | quoted-string ) ]
  ```
   
     




  




 









