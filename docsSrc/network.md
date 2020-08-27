---
id: network
title: 使用网络
---

### 配置及调用
WeTrident 推荐将服务器端的API统一管理，相比于把api接口直接零散的写入代码，配置的方式可以为后续针对接口的数据分析提供诸多便利。WeTrident中网络的配置如下: 

``` js
// modules/book/cgi/index.js
export default {
  requestBookList: {
    baseURL: 'https://www.mocky.io/',
    url: '/v2/5cedf70f300000bd1f6e97fd',
    desc: '请求书籍列表',
  }
}
```
WeTrident App中建议统一管理后台的API，每个模块强相关的后台API，都放到 `modules/$moduleName/cgi/`目录统一管理，并且通过配置都方式配置每个API需要的信息。
主要出于如下几点考虑： 
1. 统一的存放方便后期维护
2. 统一的配置格式要求每个接口要填写一些关键字段，方便以后理解。所有字段的说明见 [API配置](/WeTrident/docs/api/APIClient#api配置)： 

| 字段名 | 说明 | 是否必须 | 
| --- | --- | --- |
| desc | API的描述,可以用于调试日志/数据分析 | 是 | 
| baseURL | API的baseURL | 是 | 
| url | API的接口路径，和baseURL拼接成完成的API路径哦 | 是 | 
| mockable | 表示此接口是否直接返回mock数据 | 否
| request | API的请求结构 | 否
| response | API的响应结构，用于定义Mock的响应 | 否

定义完接口以后，我们在BookListScene中通过`APIClient`使用此接口。
``` js
// modules/book/BookListScene/BookListScene.js
export default class BookListScene extends WeBaseScene {
  // ...
  componentDidMount () {
    // 请求
    APIClient.request(CGI.requestBookList, {
      start: 0,
      pageSize: 10
    }).then(response => {
      console.log(response)
    }, error => {
      console.warning(error)
    })
  }
```


### 使用Mock
开发过程中，可能会需要再服务器端接口开发完之前开始开发前端，为了解决没有接口可用的问题，WeTrident支持了mock的功能，只需要简单的再接口配置中配置mock的返回即可，例如上面的拉去书籍列表的接口如下配置以后即可支持mock，`APIClient`发出请求以后会直接返回mock数据。response是一个数组，这个数组里面的内容随机返回，用于模拟调试失败或者多种返回数据的情况。
```javascript
// modules/book/cgi/index.js
import {AxiosMocker} from '@webank/trident'
export default {
  requestBookList: {
    baseURL: 'https://www.mocky.io/',
    url: '/v2/5185415ba171ea3a00704eed',
    desc: '请求书籍列表',

    // set true to return mock data for this api
    mockable: true,
    request: {},
    response: [
      // mock network error
      AxiosMocker.networkError(),
      // mock network timeout
      AxiosMocker.timeout(),
      // mock a normal response, it show the normal response data structure too
      AxiosMocker.success([
        {
          title: '经济学原理',
          author: '曼昆',
          coverURL: 'https://img3.doubanio.com/view/subject/l/public/s3802186.jpg',
          publishTime: '2009-4-1',
          pages: 540,
          ISBN: '9787301150894'
        },
        {
          title: '失控-全人类的最终命运和结局',
          author: '[美] 凯文·凯利 ',
          coverURL: 'https://img3.doubanio.com/view/subject/l/public/s4554820.jpg',
          publishTime: '2010-12',
          pages: 707,
          ISBN: '9787513300711'
        }
      ])
    ]
  }
}
```

在WeBookStore中，我们按照要求把`BookDetailScene`的逻辑补充完整。这里主要是完善界面和添加借阅接口的请求，这里略过，补充完以后，`BookDetailScene`内容如下： 

```js
import React, { Component } from 'react'
import { View, Text } from 'react-native'
import { AppNavigator, WeBaseScene, Toast } from '@webank/trident'
import BookDetail from './components/BookDetail'
import SimpleButton from './components/SimpleButton'
import BookDetailService from './BookDetailService'

export default class BookDetailScene extends WeBaseScene {
  static navigationOptions = ({ navigation: { state: { params = {} } } }) => ({
    headerTitle: params.title
  })

  render () {
    const bookDetail = (this.props.bookList || []).find(item => item.ISBN === this.params.ISBN)

    if (!bookDetail) {
      return null
    }
    return (
      <View>
        <BookDetail {...bookDetail} />
        <SimpleButton
          style={{
            marginTop: 8,
            paddingHorizontal: 8
          }}
          onPress={() => {
            BookDetailService.requestBorrowBook().then(response => {
              Toast.show(response.data.ret_msg)
              AppNavigator.book.ResultScene({ISBN: this.params.ISBN})
            }, error => {
              Toast.show('network error' + JSON.stringify(error))
            })
          }} title={'借阅'} />
      </View>
    )
  }
}
```

如果请求后台成功，直接跳转`ResultScene`，这里也把`ResultScene`的界面补充完成，完成以后 `ResultScene` 代码如下： 
```js
import React, { Component } from 'react'
import { View, Text, Image } from 'react-native'
import { AppNavigator, WeBaseScene } from '@webank/trident'

export default class ResultScene extends WeBaseScene {
  render () {
    const bookDetail = (this.props.bookList || []).find(item => item.ISBN === this.params.ISBN)

    return (
      <View style={[{
        flexDirection: 'column',
        justifyContent: 'flex-start',
        alignItems: 'center',
      }, this.props.style]}>
        <Image style={{
          marginTop: 80,
          marginBottom: 20,
          width: 120,
          height: 120,
        }} source={require('assets/images/success.png')} />
        <Text>{`借阅《${bookDetail.title}》成功`}</Text>
      </View>
    )
  }
}
```


### 使用Cache
除了可以发起正常请求，WeTrident的APIClient还支持了客户端的缓存。
目前的缓存支持三种模式配置缓存时间： 
1. 全局缓存时间配置
```
APIClient.setDefaultCacheMaxAgeInMs(5 * 60 * 1000)
```
2. API静态配置缓存时间
```
// modules/book/cgi/index.js 
import {AxiosMocker} from '@webank/trident'
// 接口定义
export default {
  requestBookListUseCache: {
    // 通过配置设置缓存时间
    cacheMaxAgeInMs: 60000,
    baseURL: 'https://www.mocky.io/',
    method: 'get',
    url: '/v2/5dc964632f0000760073ec4b',
    desc: '请求书籍列表',
    request: {
    },
  }
}


APIClient.request(CGI.requestBookListUseCache).then(...)
```
3. API调用动态设置缓存时间

```
APIClient.request(
  CGI.requestBookListUseCache,
  undefined,
  undefined,
  undefined,
  { cacheMaxAgeInMs: 10 }
).then(...)
```

更多用法见：[APIClient API](/WeTrident/docs/api/APIClient)
