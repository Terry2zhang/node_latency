# node_latency
a method to measure the node_latency of ray

主要是使用linux系统命令ping来进行节点间时延的测量

##### 1.打开 \src\ray\raylet\scheduling 的 cluster_resource_scheduler.cc文件

1. 添加自定义更新本地节点资源函数

​    在其中添加函数void ClusterResourceScheduler::UpdateLocalResourceInstances(const std::string &resource_name, const std::vector \<FixedPoint\> &instances)，该函数用于将时延写入到本地节点local_resources_的 custom_resources中（杨坤学长提供）

   2.添加时延测量代码

修改void ClusterResourceScheduler::FillResourceUsage(rpc::ResourcesData &resources_data) 函数，在其中添加如下代码

    std::vector<std::string> VstrIp;
    std::string strIp1 = "自定义ip1";
    std::string strIp2 = "自定义ip2";
    VstrIp.push_back(strIp1);
    VstrIp.push_back(strIp2);
    std::vector<FixedPoint> instances; 
    for (size_t i = 0; i < VstrIp.size(); i++){
    std::string strCmd = "ping -c 1 " + VstrIp[i];
    char buf[1024] = {0};  
    	FILE *pf = NULL;  
    	std::string strResult;  
    	pf = popen(strCmd.c_str(), "r");
    	while(fgets(buf, sizeof buf, pf)){  
        	strResult += buf;  
    } 
        size_t flag =0;
        std::string output ="";
        for(size_t i = 28;i<=35;i++){
            if(buf[i]=='/') {
            flag++;
            continue;}
            if(flag==1) output+=buf[i]; 
    }
    	pclose(pf); 
    	size_t n = atoi(output.c_str());
    	RAY_LOG(INFO) << "n:"<<n;
    	instances.push_back(FixedPoint(n));   
    }
    UpdateLocalResourceInstances("latency",instances);
        int64_t resource_id = string_to_int_map_.Get("latency");
        ResourceInstanceCapacities *node_instances;
        node_instances = &local_resources_.custom_resources[resource_id];
        for (size_t i = 0; i <node_instances->total.size(); i++) {
           RAY_LOG(INFO) << "latency:"<<node_instances->total[i];
        }

__***注：*** ip地址需换成自己想测量的ip

##### 2.打开 \src\ray\raylet\scheduling 的 cluster_resource_scheduler.h文件

1. 添加库函数

   #include\<cstdlib\>
   #include\<string\>
   #include\<cstdio\>
   #include\<cstring\>

2. 添加函数声明

   void UpdateLocalResourceInstances(const std::string &resource_name，const std::vector<FixedPoint> &instances);

##### 3.启动ray

- 把代码中的ray.init()换成 ray.init(address='本机IP:6379', _redis_password='5241590000000000') 

- ray start --head --resources='{"latency":40}'  --node-ip-address=本机IP

__***注：***  时延的具体值已用RAY_LOG(INFO)输出，可到 /tmp/ray/sessions_latest/logs里面的raylet.out中查看

