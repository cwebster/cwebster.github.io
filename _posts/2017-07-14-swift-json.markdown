---
layout: single
title:  "Parsing JSON with Swift"
date:   2017-07-14 00:18:23 +0700
categories: [swift, JSON, API]
---
Parsing JSON with swift. An experiment in preparation for a new app. Parse JSON feed into object defined in struct. Extensions to Struct deal with initialisation - decode of JSON string and retreiving JSON from web server API.

Utilised by:


{% highlight swift %}
    LaboratoryTest.laboratoryTests{laboratoryTests
      in
        self.laboratoryTestsDataSouce = laboratoryTests
        self.LaboratoryTestsTableView.dataSource = self
        DispatchQueue.main.async{
              self.LaboratoryTestsTableView.reloadData()
          }
     }


{% endhighlight %}


{% highlight swift %}
    
    import Foundation

    struct LaboratoryTest {
          let attributes: (analyteName: String, snomed: String?, readCode: String?)
          let id : Int
          let links: String?
          let type: String
    }

    enum SerializationError: Error {
        case missing(String)
        case invalid(String, Any)
    }


    extension LaboratoryTest: JSONDecodable {
      init?(JSON: Any) throws {
          guard let JSON = JSON as? [String: AnyObject] else { return nil }
        
          guard let analyteNameName = JSON["attributes"]?["analyte-name"] as? String else {
              return nil
          }
  
          guard let record_id = JSON["id"] as? String else {
              return nil
          }
        
          let snomodCode = JSON["attributes"]?["snomed"] ?? "Definitely Not Nil Variable"
          let readCodeCode = JSON["attributes"]?["read-code"] ?? "Definitely Not Nil Variable"
          let link = JSON["links"] ?? "Definitely Not Nil Variable" as AnyObject
          let typestr = JSON["type"] ?? "Definitely Not Nil Variable" as AnyObject

          self.attributes = (analyteName: analyteNameName, snomed: snomodCode as? String, readCode: readCodeCode as? String)
          self.id = Int(record_id)!
          self.links = link as? String
          self.type = typestr as! String
      }
    }

    extension LaboratoryTest {  
      static func laboratoryTests(completion:  @escaping (Array<Any>) -> ()) {
          let urlString = "https://api.reflabs.uk/api/v1/laboratory_tests/"
        
          let url = URL(string: urlString)
            
          URLSession.shared.dataTask(with:url!) { (data, response, error) in
              if error != nil {
                  print(error ?? "No Error")
              } else {
                  do {
                      let parsedData = try JSONSerialization.jsonObject(with: data!) as! [String:Any]
                      
                      if let labTests = parsedData["data"] as? [[String:Any]] {
                          var laboratoryTests: [LaboratoryTest] = []
                          for case let result in labTests {
                              if let laboratoryTest = try LaboratoryTest(JSON: result) {
                                  laboratoryTests.append(laboratoryTest)
                              }
                          }
                           completion(laboratoryTests)
                      }
                  } catch let error as NSError {
                      print(error )
                  }
              }
              }.resume()
      }
    }
{% endhighlight %}
