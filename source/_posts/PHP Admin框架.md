---
title: PHP Admin框架
urlname: gro0k7yi4gl818k8
date: '2022-12-21 14:01:16 +0800'
tags: []
categories: []
---

## catchAdmin

### 介绍

> 基于[thinkphp framework](http://www.thinkphp.cn/)和[element admin](https://github.com/PanJiaChen/vue-element-admin/)二次开发而成后台管理系统
> 特点：
>
> - 前后端分离
> - 基于 tp6
> - 耦合性低

#### 功能

- 用户管理 后台用户管理
- 部门管理 配置公司的部门结构，支持树形结构
- 岗位管理 配置后台用户的职务
- 菜单管理 配置系统菜单，按钮等等
- 角色管理 配置用户担当的角色，分配权限
- 数据字典 管理后台表结构
- 操作日志 后台用户操作记录
- 登录日志 后台系统用户的登录记录
- 代码生成 生成 API 端的 CURD 操作
- 敏感词 支持敏感词配置
- 附件管理 可管理上传的文件

### 安装

#### 后端安装

环境要求：

- PHP >= 7.1.0
- Mysql >= 5.5.0
- PDO Extension
- MBstring Extension
- CURL Extension
- ZIP Extension
- Composer

```bash
git clone https://gitee.com/jaguarjack/catchAdmin.git
或
composer create-project jaguarjack/catchadmin:dev-master catchAdmin

// 安装 composer 扩展
composer install --ignore-platform-reqs

/ 安装后台, 按照提示输入对应信息即可
php think catch:install

// 启动后台
php think run

```

##### nginx 配置

```nginx
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip 配置
    gzip  on;
    gzip_min_length 1k;
    gzip_comp_level 4;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript ;
    gzip_static on;
    gzip_vary on;
    gzip_buffers 8 16k;


    include /etc/nginx/conf.d/*.conf;
}
```

#### 前端安装

```bash
//配置 yarn 淘宝镜像
yarn config set registry https://registry.npm.taobao.org/

//下载项目
git clone https://github.com/yanwenwu/catch-admin-vue.git

//进入目录安装
yarn install
//安装过程中可能会出现缺少 core-js 包，安装下 yarn add core-js

//配置接口地址
//找到 vue 项目下的 .env.development 文件是配置开发环境的 API 接口地址 (实际上就是 PHP 项目的地址)

//启动项目
npm run dev

//打包项目
//打包前请先配置正是环境 API 地址。在项目的根目录下的.env.production文件配置
# just a flag
ENV = 'production'

# base api
VUE_APP_BASE_API = '正式环境的 API 地址'
//打包
npm run build:prod
```

### 项目目录

#### 目录结构

```bash
|-- app
    |-- ExceptionHandle.php // 异常捕获
    |-- service.php // 核心服务注入
|-- catch // 核心目录
|-- config
    |-- catch.php // 配置文件
|-- extend
    |-- catcher // 扩展库目录
|-- public
    |-- catch-admin // 资源目录
|-- runtime
|-- route
|-- vendor
|-- view
```

#### 核心目录

> 该目录是真正的开发目录，当然如果你不喜欢在此开发，也可以在 app 下开发，并没有什么影响。下面来说明目录结构。以 permissions 目录为例 (关于在 app 目录下进行多应用的开发, 请查看[tp 多应用开发](https://www.kancloud.cn/manual/thinkphp6_0/1297876))

```bash
|-- catch
   |-- permissions
      |-- controller //控制器
      |-- model //模型
      |-- database //数据库
         |-- migrations//表结构
         |-- seeds//数据库默认数据
      |-- request//存在表单请求，验证规则可以写在这里
      |-- module.json //保存模块信息
      |-- route.php//路由文件，路由规则需要写在里面

```

#### 公共库

> 公共库 extend\cacher 目录, 里面主要存放封装的类库。

```bash
|-- extend
   |-- catcher
      |-- base//目录存在一些基类
      |-- command//目录存在 console 命令
      |-- event//目录存在事件
      |-- exceptions//目录存放自定义异常类
      |-- traits//目录复用类库
      |-- validates//目录存在自定义的验证器
      |-- CatchAdmin.php//获取 catchAdmin 的目录信息
      |-- CatchForm.php//快速生成表单 (前后端项目无用)
      |-- CatchResponse.php//响应
      |-- Tree.php
      |-- Code.php
      |-- Utils.php
      |-- CatchQuery.php
      |-- CatchAuth.php
```

### 相关命令

#### 安装框架

```bash
php think catch:install

-r 重置数据库
```

#### 备份数据

```bash
php think backup:data <表名>,<表名>
-z 是否压缩为 zip 格式
```

#### 创建模块

```bash
php think create:module moduleName


该命令会在 catch 目录下生成模块文件夹，默认有三个文件加上若干文件夹

moduleService.php // 服务启动文件
route.php // 路由文件
module.json // 模块信息描述文件
这样基本的模块就创建完毕了。
```

#### 创建表

```bash
php think catch-migrate:run module
```

#### 创建 migration

```bash
php think catch-migrate:create
```

#### 执行 migration

```bash
php think catch-migrate:run moduleName
```

#### 创建模型

```bash
php think create:model moduleName modelName
```

#### 导出菜单

```bash
php think export permissions -p parent_id -m system
```

#### 缓存敏感词

```bash
php think cache:sensitiveWord
```

### 请求验证

> 无需在 Controller 声明验证规则在校验捕获异常，该 Request 全部封装好了。在进入 Controller 之前进行校验，将验证和 Controller 分离。保证了代码简洁和可读性。
> 定义验证规则在 `module/request/moduleRequest.php` 中。

#### 获取登录用户信息 ph

```php
$request->user()
```

#### 通用验证

> 内置验证规则，参考 [内置规则](https://www.kancloud.cn/manual/thinkphp6_0/1037629)
> 提示信息，参考 [提示信息](https://www.kancloud.cn/manual/thinkphp6_0/1037626)

```php
<?php
namespace catchAdmin\login\request;

use catcher\base\CatchRequest;
//继承自 CatchRequest
class LoginRequest extends CatchRequest
{
    protected $needCreatorId = false;//属性是否添加创建人 id，默认 true
    protected $batch = false;//是否批量验证，默认 false

    //在这里定义验证规则
    protected function rules(): array
    {
        // TODO: Implement rules() method.
        return [
            'email|用户名'    => 'email',
            'password|密码'  => 'require',
           // 'captcha|验证码' => 'require|captcha'
        ];
      //常见内置规则：require、number、integer、float、boolean、bool、email、array
      //accepted（验证某个字段是否为为 yes, on, 或是 1。这在确认"服务条款"是否同意时很有用）
      //date、alpha（纯字母）、alphaNum（字母和数字）、alphaDash（字母、数字、-、_）
      //chs（只能汉字）、chsAlpha（汉字、字母）、chsAlphaNum（汉字、字母、数字）
      //chsDash（汉字、字母、数字、_、-）
      //cntrl（验证某个字段的值只能是控制字符（换行、缩进、空格））
      //graph（只能是可打印字符（空格除外）
      //lower（小写字母）、upper（大写字母）
      //space（空白字符）、xdigit（十六进制字符串）
      //activeUrl（有效的域名或者IP）、url、ip
      //dateFormat:y-m-d（指定格式的日期）
      //mobile、idCard、macAddr（mac 地址）、zip（邮编）
      //in:1,2,3（在某个范围）、notIn:1,2,3（不在某个区间）、between:1,10（在某个区间）、notBetween


    }

    protected function message(): array
    {
        // TODO: Implement message() method.
        return [];

    }
}

```

#### 自定义验证

> 在 `extend/catcher/validates/自定义规则`。
> 需要实现 `ValidateInterface`接口。
> 写完注册到 `config\catch.php `中

##### 定义

```php
<?php
declare(strict_types=1);

namespace catcher\validates;

class Sometimes implements ValidateInterface
{

    public function type(): string
    {
        return 'sometimes';
    }

    public function verify($value): bool
    {
        if ($value) {
            return true;
        }

        return false;
    }

    public function message(): string
    {
        return '';
    }
}

```

##### 注册

```php
'validates' => [
    \catcher\validates\Sometimes::class,
  ],
```

### 模型

> 在 `module/model/<模型名>Model.php`中定义。
> 需继承 `catcher\base\CatchModel`模型

#### 模型定义

```php
class Attachments extends CatchModel
{
    protected $name = '表名';

    protected $field = [//字段名
            'id', //
            'path', // 附件存储路径
            'url', // 资源地址
            'mime_type', // 资源mimeType
            'file_ext', // 资源后缀
            'file_size', // 资源大小
            'filename', // 资源名称
            'driver', // local,oss,qcloud,qiniu
            'created_at', // 创建时间
            'updated_at', // 更新时间
            'deleted_at', // 删除时间
    ];
}
```

#### catchModel

```php
<?php
declare(strict_types=1);

namespace catcher\base;

use catcher\CatchQuery;//使用了 CatchQuery
use catcher\traits\db\BaseOptionsTrait;
use catcher\traits\db\RewriteTrait;
use catcher\traits\db\WithTrait;
use think\model\concern\SoftDelete;
use catcher\traits\db\ScopeTrait;

/**
 *
 * @mixin CatchQuery
 * Class CatchModel
 * @package catcher\base
 */
abstract class CatchModel extends \think\Model
{
    use SoftDelete, BaseOptionsTrait, ScopeTrait, RewriteTrait, WithTrait;

    protected $createTime = 'created_at';//创建时间

    protected $updateTime = 'updated_at';//更新时间

    protected $deleteTime = 'deleted_at';//删除时间

    protected $defaultSoftDelete = 0;//是否开启软删除

    protected $autoWriteTimestamp = true;//自动写入时间戳

    // 分页 Limit
    public const LIMIT = 10;
    // 开启
    public const ENABLE = 1;
    // 禁用
    public const DISABLE = 2;

    /**
     * 是否有 field
     *
     * @time 2020年11月23日
     * @param string $field
     * @return bool
     */
    public function hasField(string $field)
    {
        return property_exists($this, 'field') && in_array($field, $this->field);
    }

    public function __construct(array $data = [])
    {
        parent::__construct($data);


        if (method_exists($this, 'autoWithRelation')) {
            $this->autoWithRelation();
        }
    }
}

```

#### catchQuery

```php
<?php
declare(strict_types=1);

namespace catcher;

use catcher\base\CatchModel;
use think\db\Query;
use think\helper\Str;
use think\Paginator;

class CatchQuery extends Query
{
  /**
   *
   * @time 2020年01月1  // model Join 的模型
  // oinField join 模型的字段
 // currentJoinField 当前模型的关联的字段
  // field 需要 join 的模型查询的字段3日
   * @param mixed $model
   * @param string $joinField
   * @param string $currentJoinField
   * @param array $field
   * @param string $type
   * @param array $bind
   * @return CatchQuery
   */
    public function catchJoin($model, string $joinField, string $currentJoinField, array $field = [], string $type = 'INNER', array $bind = []): CatchQuery
    {
        $tableAlias = null;

        if (is_string($model)) {
            $table = app($model)->getTable();
        } else {
            list($model, $tableAlias) = $model;
            $table = app($model)->getTable();
        }

        // 合并字段
        $this->options['field'] = array_merge($this->options['field'] ?? [], array_map(function ($value) use ($table, $tableAlias) {
          return ($tableAlias ? : $table) . '.' . $value;
        }, $field));

        return $this->join($tableAlias ? sprintf('%s %s', $table, $tableAlias) : $table

            , sprintf('%s.%s=%s.%s', $tableAlias ? $tableAlias : $table, $joinField, $this->getAlias(), $currentJoinField), $type, $bind);
    }

  /**
   *
   * @time 2020年01月13日
   * @param mixed $model
   * @param string $joinField
   * @param string $currentJoinField
   * @param array $field
   * @param array $bind
   * @return CatchQuery
   */
    public function catchLeftJoin($model, string $joinField, string $currentJoinField, array $field = [], array $bind = []): CatchQuery
    {
        return $this->catchJoin($model, $joinField,  $currentJoinField,  $field,'LEFT', $bind);
    }

  /**
   *
   * @time 2020年01月13日
   * @param mixed $model
   * @param string $joinField
   * @param string $currentJoinField
   * @param array $field
   * @param array $bind
   * @return CatchQuery
   */
    public function catchRightJoin($model, string $joinField, string $currentJoinField, array $field = [], array $bind = []): CatchQuery
    {
        return $this->catchJoin($model, $joinField,  $currentJoinField, $field,'RIGHT', $bind);
    }

  /**
   * rewrite
   *  // 这个方法很常用
  // 例如你表里有 10 个字段，但是偏偏有一字段你不需要，那么这个方法可以帮你过滤掉该字段
  // ！注意 是主表的字段
   * @time 2020年01月13日
   * @param array|string $field
   * @param bool $needAlias
   * @return $this|Query
   */
    public function withoutField($field, bool $needAlias = false)
    {
      if (empty($field)) {
          return $this;
      }

      if (is_string($field)) {
          $field = array_map('trim', explode(',', $field));
      }

      // 过滤软删除字段
      $field[] = $this->model->getDeleteAtField();

      // 字段排除
      $fields = $this->getTableFields();
      $field  = $fields ? array_diff($fields, $field) : $field;

      if (isset($this->options['field'])) {
          $field = array_merge((array) $this->options['field'], $field);
      }

      $this->options['field'] = array_unique($field);

      if ($needAlias) {
          $alias = $this->getAlias();
          $this->options['field'] = array_map(function ($field) use ($alias) {
          return $alias . '.' . $field;
        }, $this->options['field']);
      }

      return $this;
    }

    /**
     * 调用它 配合搜索器完美实现搜索
     * @time 2020年01月13日
     * @param array $params
     * @return CatchQuery
     */
    public function catchSearch(array $params = []): CatchQuery
    {
        $params = empty($params) ? \request()->param() : $params;

        if (empty($params)) {
            return $this;
        }

        foreach ($params as $field => $value) {
            $method = 'search' . Str::studly($field) . 'Attr';
            // value in [null, '']
            if ($value !== null && $value !== '' && method_exists($this->model, $method)) {
                $this->model->$method($this, $value, $params);
            }
        }

        return $this;
    }

    /**
     * 快速搜索
     *
     * @param array $params
     * @return Query
     */
    public function quickSearch(array $params = []): Query
    {
        $requestParams = \request()->param();

        if (empty($params) && empty($requestParams)) {
            return $this;
        }

        foreach ($requestParams as $field => $value) {
            if (isset($params[$field])) {
                // ['>', value] || value
                if (is_array($params[$field])) {
                    $this->where($field, $params[$field][0], $params[$field][1]);
                } else {
                    $this->where($field, $value);
                }
            } else {
                // 区间范围 start_数据库字段 & end_数据库字段
                $startPos = strpos($field, 'start_');
                if ($startPos === 0) {
                    $this->where(str_replace('start_','', $field), '>=', strtotime($value));
                }
                $endPos = strpos($field, 'end_');
                if ($endPos === 0) {
                    $this->where(str_replace('end_', '', $field), '<=', strtotime($value));
                }
                // 模糊搜索
                if (Str::contains($field, 'like')) {
                    [$operate, $field] = explode('_', $field);
                    if ($operate === 'like') {
                        $this->whereLike($field, $value);
                    } else if ($operate === '%like') {
                        $this->whereLeftLike($field, $value);
                    } else {
                        $this->whereRightLike($field, $value);
                    }
                }

                // = 值搜索
                if ($value || is_numeric($value)) {
                    if ($field != 'page' && $field != 'limit' && $startPos !== 0 && $endPos !== 0 && $operate !== 'like' && $operate !== '%like') {
                        $this->where($field, $value);
                    }
                }
            }
        }

        return $this;
    }

  /**
   *
   * @time 2020年01月13日
   * @return mixed
   */
    public function getAlias()
    {
      return isset($this->options['alias']) ? $this->options['alias'][$this->getTable()] : $this->getTable();
    }

  /**
   * rewrite
   *获取 alias 字段
   * @time 2020年01月13日
   * @param string $field
   * @param mixed $condition
   * @param string $option
   * @param string $logic
   * @return Query
   */
    public function whereLike(string $field, $condition, string $logic = 'AND', string $option = 'both'): Query
    {
        switch ($option) {
          case 'both':
              $condition = '%' . $condition . '%';
              break;
          case 'left':
              $condition = '%' . $condition;
              break;
          default:
              $condition .= '%';
        }

        if (strpos($field, '.') === false) {
            $field = $this->getAlias() . '.' . $field;
        }

        return parent::whereLike($field, $condition, $logic);
    }

    /**
     * @param string $field
     * @param $condition
     * @param string $logic
     * @return Query
     */
    public function whereLeftLike(string $field, $condition, string $logic = 'AND'): Query
    {
        return $this->where($field, $condition, $logic, 'left');
    }

    /**
     * @param string $field
     * @param $condition
     * @param string $logic
     * @return Query
     */
    public function whereRightLike(string $field, $condition, string $logic = 'AND'): Query
    {
        return $this->where($field, $condition, $logic, 'right');
    }

  /**
   * 额外的字段
   *
   * @time 2020年01月13日
   * @param $fields
   * @return CatchQuery
   */
    public function addFields($fields): CatchQuery
    {
        if (is_string($fields)) {
            $this->options['field'][] = $fields;

            return $this;
        }

        $this->options['field'] = array_merge($this->options['field'], $fields);

        return $this;
    }
  	//分页
    public function paginate($listRows = null, $simple = false): Paginator
    {
        if (!$listRows) {
           $limit = \request()->param('limit');

           $listRows = $limit ? : CatchModel::LIMIT;
        }

        return parent::paginate($listRows, $simple); // TODO: Change the autogenerated stub
    }


    /**
     * 默认排序
     *
     * @time 2020年06月17日
     * @param string $order
     * @return $this
     */
    public function catchOrder(string $order = 'desc'): CatchQuery
    {
        if (in_array('sort', array_keys($this->getFields()))) {
            $this->order($this->getTable() . '.sort', $order);
        }

        if (in_array('weight', array_keys($this->getFields()))) {
            $this->order($this->getTable() . '.weight', $order);
        }

        $this->order($this->getTable() . '.' . $this->getPk(), $order);

        return $this;
    }

    /**
     * 新增 Select 子查询
     *
     * @time 2020年06月17日
     * @param callable $callable
     * @param string $as
     * @return $this
     */
    public function  addSelectSub(callable $callable, string $as): CatchQuery
    {
        $this->field(sprintf('%s as %s', $callable()->buildSql(), $as));

        return $this;
    }

    /**
     * 字段增加
     *
     * @time 2020年11月04日
     * @param $field
     * @param int $amount
     * @return int
     *@throws \think\db\exception\DbException
     */
    public function increment($field, int $amount = 1): int
    {
        return $this->inc($field, $amount)->update();
    }

    /**
     * 字段减少
     *
     * @time 2020年11月04日
     * @param $field
     * @param int $amount
     * @return int
     *@throws \think\db\exception\DbException
     */
    public function decrement($field, int $amount = 1): int
    {
        return $this->dec($field, $amount)->update();
    }
}

```

### 权限

> - GET 请求是默认不经过权限控制，如果需要验证权限
>   - 需要在方法注释中加入 @CatchAuth 标识
> - 超级管理员不经过权限控制,后台默认安装的用户

#### 数据权限

> 关于数据权限的概念，很简单，就是要标记数据的所有者。所以
> 如果你需要数据权限的时候，那么表结构需要默认的 `creator_id` 字段，用来标记数据的所有者。
> 一旦使用了数据权限，那么可以使用 CatchRequest,使用它可以无缝获取 `creator_id`，这是无感知的。

```php
//获取 creator_id
$request->param()
或
$request->post()
```

#### 使用

> CatchAdmin 封装了可用 trait 来帮助开发者处理数据权限数据，引入 trait
> 在模型中使用 `dataRange` 方法，该方法接受一个 roles 对象数组，如果不传，则获取当前登录用户的角色组。以用户列表为例

```php
use catchAdmin\permissions\model\DataRangScopeTrait
```

```php
$this->dataRange()
  ->withoutField(['updated_at'], true)
  ->catchSearch()
  ->catchLeftJoin(Department::class, 'id', 'department_id', ['department_name'])
  ->order($this->aliasField('id'), 'desc')
  ->paginate();
```

### 响应

> 在控制器中使用 `CatchRespons ` 类返回结果数据

```php
<?php

namespace catchAdmin\cms\controller;

use catcher\base\CatchRequest as Request;
use catcher\CatchResponse;//使用 CatchResponse
use catcher\base\CatchController;
use catchAdmin\cms\model\Articles as articlesModel;

class Articles extends CatchController
{
    protected $articlesModel;

    public function __construct(ArticlesModel $articlesModel)
    {
        $this->articlesModel = $articlesModel;//初始化加载模型
    }

    /**
     * 列表
     * @time 2020年12月27日 19:40
     * @param Request $request
     */
    public function index(Request $request) : \think\Response
    {
        return CatchResponse::paginate($this->articlesModel->getList());
    }

    /**
     * 保存信息
     * @time 2020年12月27日 19:40
     * @param Request $request
     */
    public function save(Request $request) : \think\Response
    {
        return CatchResponse::success($this->articlesModel->storeBy($request->post()));
    }

    /**
     * 读取
     * @time 2020年12月27日 19:40
     * @param $id
     */
    public function read($id) : \think\Response
    {
        return CatchResponse::success($this->articlesModel->findBy($id));
    }

    /**
     * 更新
     * @time 2020年12月27日 19:40
     * @param Request $request
     * @param $id
     */
    public function update(Request $request, $id) : \think\Response
    {
        return CatchResponse::success($this->articlesModel->updateBy($id, $request->post()));
    }

    /**
     * 删除
     * @time 2020年12月27日 19:40
     * @param $id
     */
    public function delete($id) : \think\Response
    {
        return CatchResponse::success($this->articlesModel->deleteBy($id));
    }
}
```

#### CatchResponse

```php
<?php
declare(strict_types=1);

namespace catcher;

use think\Paginator;
use think\response\Json;

class CatchResponse
{
  /**
   * 成功的响应
   *
   * @time 2019年12月02日
   * @param array $data
   * @param $msg
   * @param int $code
   * @return Json
   */
  public static function success($data = [], $msg = 'success', $code = Code::SUCCESS): Json
  {
        return json([
          'code'    => $code,
          'message' => $msg,
          'data'    => $data,
        ]);
  }

  /**
   * 分页
   *
   * @time 2019年12月06日
   * @param Paginator $list
   * @return
   */
  public static function paginate(Paginator $list)//接受 Paginator 对象
  {
        return json([
          'code'    => Code::SUCCESS,
          'message' => 'success',
          'count'   => $list->total(),
          'current' => $list->currentPage(),
          'limit'   => $list->listRows(),
          'data'    => $list->getCollection(),
        ]);
  }

  /**
   * 错误的响应
   *
   * @time 2019年12月02日
   * @param string $msg
   * @param int $code
   * @return Json
   */
  public static function fail($msg = '', $code = Code::FAILED): Json
  {
        return json([
            'code' => $code,
            'message'  => $msg,
        ]);
  }
}

```
