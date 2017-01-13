# ELB-with-proxy-testing

1. Create ELB first 
   you can colne the repo 
   
  ```
   # git clone ssh://zzhao@code.engineering.redhat.com:22/openshift-misc
   # cd openshift-misc/v3-launch-templates/functionality-testing/aos-33
   # ansible-playbook extra-ansible/create_aws_elb.yaml -v -e aws_elb_enable=false -e new_aws_elb=true
   ```
2. setup the openshift env with jenkins using the following 

```    
vm_type: m3.medium
api_lb_address: qe-api-01101539-1479489583.us-east-1.elb.amazonaws.com
infra_lb_address: "52.72.181.46"
master_elbname: qe-api-01101539
infra_elbname: qe-infra-01101539
```    
 3. config the ELB
 
 ```
   $ aws elb create-load-balancer-policy --load-balancer-name qe-infra-01101539 --policy-name infra-lb-ProxyProtocol-policy --policy-type-name ProxyProtocolPolicyType --policy-attributes AttributeName=ProxyProtocol,AttributeValue=true
   $ aws elb set-load-balancer-policies-for-backend-server --load-balancer-name qe-infra-01101539 --instance-port 443 --policy-names infra-lb-ProxyProtocol-policy
   $ aws elb set-load-balancer-policies-for-backend-server --load-balancer-name qe-infra-01101539 --instance-port 80 --policy-names infra-lb-ProxyProtocol-policy
aws elb delete-load-balancer-listeners --load-balancer-name qe-infra-01101539 --load-balancer-ports 80
   $ aws elb create-load-balancer-listeners --load-balancer-name qe-infra-01101539 --listeners Protocol=TCP,LoadBalancerPort=80,InstanceProtocol=TCP,InstancePort=80
```   
 4. enable the proxy in haproxy router pod
 
    `oc env dc ROUTER_USE_PROXY_PROTOCOL=true`
    
  5. Create the test app
    
    ```
    oc create -f https://raw.githubusercontent.com/openshift-qe/v3-testfiles/master/routing/header-test/dc.json
    oc create -f https://raw.githubusercontent.com/openshift-qe/v3-testfiles/master/routing/header-test/insecure-service.json
    oc expose svc header-test-insecure
    ```
  6. Check the header origin ip
   
   ```
   curl header-test-insecure-zzhao.0113-bqm.qe.rhcloud.com
   <pre>
  user-agent: curl/7.40.0
  host: header-test-insecure-zzhao.0113-bqm.qe.rhcloud.com
  accept: */*
  x-forwarded-host: header-test-insecure-zzhao.0113-bqm.qe.rhcloud.com
  x-forwarded-port: 80
  x-forwarded-proto: http
  forwarded: for=119.254.120.72;host=header-test-insecure-zzhao.0113-bqm.qe.rhcloud.com;proto=http
  x-forwarded-for: 119.254.120.72
</pre>
 #### edge route ###
 curl https://edge1-zzhao.0113-bqm.qe.rhcloud.com -k
<pre>
  user-agent: curl/7.40.0
  host: edge1-zzhao.0113-bqm.qe.rhcloud.com
  accept: */*
  x-forwarded-host: edge1-zzhao.0113-bqm.qe.rhcloud.com
  x-forwarded-port: 443
  x-forwarded-proto: https
  forwarded: for=119.254.120.72;host=edge1-zzhao.0113-bqm.qe.rhcloud.com;proto=https
  x-forwarded-for: 119.254.120.72
</pre>
```
