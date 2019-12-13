# 一、CMPedometer(计步器)
##  1、介绍  
   -   统计某段时间内用户步数、距离信息，用户爬了多次级楼梯，处理用户运动与健康的信息<br>
   -   iOS8.0以后使用
   
   
##  2、使用
- 在对应的类中导入CoreMotion库 ：` <CoreMotion/CoreMotion.h>`
   
 ```
 #import “ViewController.h"
 #import <CoreMotion/CoreMotion.h
 @interface ViewController ()
 @property(nonatomic,strong) CMPedometer *pedometer; 
 @end  

 ```

- 在ViewDidLoad方法里面实现

 
 ```
     //初始化
    self.pedometer = [[CMPedometer alloc]init];
    //判断记步功能权限
    if ([CMPedometer isStepCountingAvailable]) {
        // date 是从什么时候开始
      [self.stepCounter startPedometerUpdatesFromDate:date withHandler:^(CMPedometerData * _Nullable pedometerData, NSError * _Nullable error) {
         if (error) {
                NSLog(@"error====%@",error);
            }else {                                                                                                                                                                                                                    
                NSLog(@“一共走了%@步数",pedometerData.numberOfSteps);
                NSLog(@"距离====%@",pedometerData.distance);
            }
    }];
    
 ```
    
- 注：需要在`info.plist`文件中添加权限，并描述：
	
	```
	//运动与健康
      NSMotionUsageDescription
	```
	
	
# 二、google地图定位
## 1、介绍
  - 参考Google地图开发者中心：[https://developers.google.com/](https://developers.google.com/maps/)
  
  
## 2、使用


### 地图


- 在需要显示GoogleMap的页面，引入库

	```
	<GoogleMaps/GoogleMaps.h>
	```

- 注册GoogleMap密钥
 
					 
	```
	[GMSServices provideAPIKey:@“***********”]
	```


- 实例化GoogleMap并显示在界面上


	```
 GMSCameraPosition *camera = [GMSCameraPosition cameraWithLatitude:-33.86
                                                            longitude:151.20
                                                                 zoom:6];
 mapView_ = [GMSMapView mapWithFrame:CGRectMake(0, 0, WIDTH, HEIGHT/5*3) camera:camera];
 [self.view addSubview:mapView_];

	```
	
	
### 定位


- 定位调用头文件以及遵守协议并声明

	```
	#import "ViewController.h"
	#import <GoogleMaps/GoogleMaps.h>
	#import <CoreMotion/CoreMotion.h>
	
	@interface SBTrackerRecord ()<CLLocationManagerDelegate>
	
	@property (nonatomic, strong) CLLocationManager *locationManager;
	
	@end
	
	```

- 设置定位管理器获取定位权限，以及设置定位相关属性

	```
    //定位管理器
    _locationManager=[[CLLocationManager alloc]init];
    //如果没有授权则请求用户授权
    if ([CLLocationManager authorizationStatus]==kCLAuthorizationStatusNotDetermined){
        [_locationManager requestWhenInUseAuthorization];
    }else if([CLLocationManager authorizationStatus]==kCLAuthorizationStatusAuthorizedWhenInUse){
         //设置代理
         _locationManager.delegate=self;
         //设置定位精度
         // kCLLocationAccuracyBestForNavigation // 最适合导航
         // kCLLocationAccuracyBest; // 最好的
         // kCLLocationAccuracyNearestTenMeters; // 附近10米
         // kCLLocationAccuracyHundredMeters; // 100米
         // kCLLocationAccuracyKilometer; // 1000米
         // kCLLocationAccuracyThreeKilometers; // 3000米
        _locationManager.desiredAccuracy=kCLLocationAccuracyBest;
        //定位频率,每隔多少米定位一次
        _locationManager.distanceFilter=10.0;//十米定位一次
        //启动跟踪定位
       // [_locationManager startUpdatingLocation];
        
        //停止跟踪定位
              //[_locationManager stopUpdatingLocation];
	
	
        //开启一次性定位
              //[_locationManager requestLocation];
	
    } 
	
	```

- delegate的回调方法，可以获取到定位的经纬度

	```
	- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations
	{
	
	     //    处理定位信息
	     //    火星文转地理位置
        //    计算两点的距离
        //    地图上画线
    	     //    .......
	     		
	}
	
	```

-   注：需要在`info.plist`中，添加权限，并描述


	```
    //只能用于应用在前台时需要获取用户位置的权限说明
    NSLocationWhenInUseUsageDescription
    
    // 用于应用在后台时需要获取用户位置的权限说明
    NSLocationAlwaysAndWhenInUseUsageDescription 
    
    //用于获取用户位置信息的权限说明
    NSLocationUsageDescription
    
    //iOS11之前，应用在后台时需要获取位置权限说明,需要跟NSLocationAlwaysAndWhenInUseUsageDescription 一起添加，
    //iOS11 之后，只添加NSLocationAlwaysAndWhenInUseUsageDescription就可以了
    NSLocationAlwaysUsageDescription
	
	```





# 3、CMPedometer（计步器）跟定位结合使用


-  在viewDidLoad中，开启计步器，用户需要同意授权获取用户的健康与运动
- 当检测到手机有进行移动，会进入回调方法


	```
	startPedometerUpdatesFromDate:(NSDate *)start withHandler:(CMPedometerHandler)handler
	```

- 在计步器的回调中，进行判断，是否跟上次回调的距离步数一样，如果不一样，则进行一次性定位，
- 定位成功后，在地图上绘制路线
