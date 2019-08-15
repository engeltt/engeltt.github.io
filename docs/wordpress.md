跨多版本升级wordpress（从4.9.5到5.2.2版本）
* 下载最新版wordpress，除了wp-config.php和wp-content，其余目录和文件均替换为最新版的
* 删除页面和文章里面被嵌入的异常js代码，此时发现文章有字数限制，超字数无法更新文章
    * mysql配置max_allowed_packet=20M（原本只有1M，数据包太大文章太长就更新数据库不成功了，也就更新文章不成功了）
* 安装安全插件，此时发现无法安装插件，提示无法创建目录
    * 修改新上传的wordpress目录所有者为php-fpm的用户www-data
    * 修改wp-content目录权限为755
    * 配置安全插件，开启安全防护
