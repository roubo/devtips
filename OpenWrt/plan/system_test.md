# 路由器建立一套系统检测机制

## 组成

      系统检测lua api ＋ 检测数据 ＋ 前端调用呈现 ＋ 后端定时检测 ＋ 数据处理

## 系统检测api

      以testXXX的格式来实现测试执行接口, 该类接口可以阻塞执行，不可重入
      以getXXX的格式来实现对某一项测试的结果获取

## 数据

      test和get接口通过uci文本数据库进行交互结果
      所有测试数据都保留在uci文本数据库中，数据处理接口只关心该数据库

## 前端调用

      test和get接口封装成http接口，供前端调用

## 后端定时

      通过crond定时调用api

## 数据处理

      对数据进行格式化，本地分析，上传
