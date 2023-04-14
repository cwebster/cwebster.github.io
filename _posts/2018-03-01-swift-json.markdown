---
layout: single
title:  "Parsing JSON with Swift"
date:   2018-03-01 00:18:23 +0700
categories: [swift, JSON, API]
---
Parsing JSON with swift. Further experiments using Alamofire [https://github.com/Alamofire/Alamofire](Alamofire) and  [https://github.com/alibaba/HandyJSON](HandJSON)

For a Model representation for type T which conforms to the HandJSON protocol

{% highlight swift %}
struct AuthenticationPayload : HandyJSON
{
    init() {}
    var token:String?
    var expiry:Int?
    var client:String?
    var user:User?
    var roles:[Role]?
    var laboratory:ApiLaboratory?
}
{% endhighlight %}

{% highlight swift %}
import Foundation
import Alamofire
import HandyJSON

protocol NetworkService {
    
}
extension NetworkService {
    
    func request<T:HandyJSON>(model: T, endpoint: Endpoint, httpMethod: HTTPMethod, parameters: Parameters , completionHandler: @escaping (T, Int?) -> Void) {
        
        let headers: HTTPHeaders = [
            "client": UserDefaults.standard.string(forKey: "clientID_preference")!
        ]
        
        let request = Alamofire.request(endpoint,  method: httpMethod, parameters: parameters, encoding: JSONEncoding.default, headers: headers)
            
            .validate(statusCode: 200..<300)
            .validate(contentType: ["application/json"])
            .responseString { response in
                let statusCode = response.response?.statusCode
                debugPrint(response)
                if let data = response.data, let utf8Text = String(data: data, encoding: .utf8) {
                    if let object = T.deserialize(from: utf8Text){
                        completionHandler(object, statusCode)
                    }
                }
        }
        debugPrint(request)
    }
}

{% endhighlight %}

conforming to this base request protocol allows somethinglike this

{% highlight swift %}
    
protocol LogInService:NetworkService {
    var username:String  { get set }
    var password:String  { get set }
    var clientID:String { get set }
}

extension LogInService {
    
    func loginUser(username:String, password:String, completionHandler: @escaping (User, Bool) -> Void) {

        var success = false
        
        let parameters: Parameters = [
            "password": password,
            "email": username,
            "client": clientID
        ]
        
        typealias authentication = (AuthenticationPayload, Int?)
        
        let payload = AuthenticationPayload()
        
        self.request(model: payload, endpoint: API.LOGIN_END_POINT, httpMethod: HTTPMethod.post, parameters: parameters, completionHandler: { authenticationPayLoad, statusCode in
            if statusCode == 200 {
                print(authenticationPayLoad.token!)
                var user = authenticationPayLoad.user
                user?.accessToken = authenticationPayLoad.token!
                user?.clientID = self.clientID
                success = true
                
                completionHandler(user!, success)
            }
        })
    }
}
{% endhighlight %}

which can be called as:
{% highlight swift %}
        loggedInUser?.loginUser(username: self.usernameField.text!, password: self.passwordField.text!, completionHandler: { user, statusCode in
            DispatchQueue.main.async {
                let viewController:UIViewController = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "SecondViewController") as! UIViewController
                self.present(viewController, animated: true, completion: nil)
            }
        })
{% endhighlight %}
