# pv、uv...

```php
// 获取用户信息
$ip = $_SERVER['REMOTE_ADDR'];

// 写入文件
file_put_contents('record.txt',$ip . "\r\n",FILE_APPEND);

// 读取文件
$info = file('record.txt');

// 总访问次数 -- 总行数
$visits = count($info);

// 当前用户的访问次数  就是当前ip出现的次数
$ip_cisits = 0;
foreach($info as $each_id){
	// if($each_id === $ip."\r\n"){
	if(trim($each_id) === $ip){
		$ip_cisits++;
	}
}

// 数组去重
$unique_ips = [];
$user_visit = 0;
foreach($info as $each_id){
	if(!in_array(trim($each_id),$unique_ips)){
		$unique_ips[] = trim($each_id);

		if($ip == trim($each_id)){
			$user_visit = count($unique_ips);
		}
	}
}

$unique_ips_count = count($unique_ips);

echo "总访问{$visits}";
echo "当前第{$ip_cisits}次访问";
echo "当前共有{$unique_ips_count}个用户";
echo "你是第{$user_visit}个访问用户";

```