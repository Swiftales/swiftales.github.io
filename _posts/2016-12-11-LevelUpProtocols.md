---
layout: post
title: Level up on protocols, associatedtypes and extensions.
---

In the [previous](https://swiftales.github.io/ProtocolsAndAssociatedtype/) blog post, we learnt about basic use of `associatedtype`. In this blog post, we will see some advance use of `protocol`, `associatedtype` and `extension` by building a generic `UITableview` datasource.

## Lets get started

How many times we get a requirement to show a simple list of data in a `UITableview` in single section. Sometimes we need to do this multiple times in one application. Just because we have different types of data sets, we have to implement `UITableView` datasource again and agian but the implementation usually has same logic.

Consider we need to show a list of `Person` in a table view. Below is the pseudo-implementation.

```swift
//Datasource setup to to show Person
var persons: [Person]

func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
 func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return persons.count
    }
    
 func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "CellID", for: indexPath) 
        cell.confgiure(forPerson person: persons[indexPath.row])
        return cell
    }
```

Now at some other place, wee need to show a list of department. The logic to feed our tableView is same but still, we need to write that logic again with different data type i.e. `Department`.

```swift
//Datasource setup to show Department
var departments: [Department]

func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
 func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return departments.count
    }
    
 func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "CellID", for: indexPath) 
        cell.confgiure(forDepartment department: departments[indexPath.row])
        return cell
    }
```


So the idea behind this generic `UITableView` datasource is to put this same logic at one place which can be reused for different types of data.


Lets start with creating a simple `protocol` which denotes that the conforming data type can be used to populate a `UITableViewCell`.

Pardon my naming conventions. I am not good in naming anything 😄.

```swift
protocol TableViewCellDataRepresentable {}
```
Now lets create a `protocol`, which denotes that the conforming types can decorate a specific `UITableViewCell` with a specific `TableViewCellDataRepresentable`. 

**Note:** since the decorator does not know the actual type of `UITableViewCell` and `TableViewCellDataRepresentable`, so we need to use a placeholder type. `associatedtype` can serve this purpose well.

```swift
protocol TableViewCellDecorator {
    associatedtype DataType: TableViewCellDataRepresentable
    associatedtype CellType: UITableViewCell
    static func update(tableViewCell cell: CellType, withData data: DataType)
}
```

So the conforming type will define its `DataType` and `CellType` and implement the logic to decorate the cell accordingly.

Now lets begin implementation of our generic `UITableView` datasource. 

Try to stay with me, its a little bit complex to keep up.

```swift
class TableViewHandler<T: TableViewCellDecorator>: NSObject, UITableViewDataSource {
    
    fileprivate(set) var dataSet: [T.DataType]
    private(set) weak var tableView: UITableView!
    let reuseIdentifiers: (IndexPath) -> String
    
    init(withTableView tableView: UITableView, dataSet: [T.DataType], reuseIdentifiers: @escaping (IndexPath) -> String) {
        self.tableView = tableView
        self.dataSet = dataSet
        self.reuseIdentifiers = reuseIdentifiers
        super.init()
        self.tableView.dataSource = self
    }
    
    func reloadWithData(data: [T.DataType]) {
        dataSet = data
        tableView.reloadData()
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return dataSet.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(withIdentifier: reuseIdentifiers(indexPath), for: indexPath) as? T.CellType else {
           return tableView.dequeueReusableCell(withIdentifier: reuseIdentifiers(indexPath), for: indexPath)
        }
        T.update(tableViewCell: cell, withData: dataSet[indexPath.row])
        return cell
    }
}
```
We created a generic `TableViewHandler` whose genric type `T` should conform to `TableViewCellDecorator`. 

Next we decalared a `dataSet` whose type is `T's` `DataType`. That is, `dataSet` will be collection of `TableViewCellDataRepresentable` whose actual type will be known at the time of declaring and intialising our `TableViewHandler` object.


This kind of coupling ensures that we wont be able to pass wrong data type and also there is no type casting involved. Everything will be known at compile time itslef.

It also has a closure property names `reuseIdentifiers`, in which the client can pass cellIdentifiers based on indexpath.

Finally, in tableView's datasoure method:
- We are returning `dataSet` count as `numberOfRows`
- Next in `cellForRowAtIndexPath`, we are getting the cell identifier by invoking `reuseIdentifiers` closure and also making sure that tableView returns a cell which our `T` i.e. `TableViewCellDecorator` can decorate.

## Example
Its time to see it in action with an example.

First we will do basic setup by creating a cell and `UIViewController` with tableView.

The is the layout of my cell, which has three labels horizontally aligned inside a stack view.

![Image alt](/assets/posts/Deep_Dive_Protocols/cellLayout.png "Cell")

I have named it.......`ThreeLabelTableViewCell` 😛. Again, pardon my naming.

There is nothing in the implementation of `ThreeLabelTableViewCell`.

This is how my Storyboard looks like. Nothing extra ordinary.

![Image alt](/assets/posts/Deep_Dive_Protocols/storyboard.png "Storyboard")

Now lets create our modal.

```swift
struct Person {
    let firstName: String
    let lastName: String
    var age: Int
    var isVIP: Bool
}

extension Person: TableViewCellDataRepresentable {}

extension Person { 
    static let vishal = Person(firstName: "Vishal Singh", lastName: "Panwar", age: 28, isVIP: false /*yeah, no VIP flag for me, I like to keep myself low ✌ */)
    static let bruce = Person(firstName: "Bruce", lastName: "Wayne", age: 35, isVIP: true)
    static let john = Person(firstName: "John", lastName: "Cena", age: 30, isVIP: true)
    static let tony = Person(firstName: "Tony", lastName: "Stark", age: 35, isVIP: true)
    static let peter = Person(firstName: "Peter", lastName: "Parker", age: 20, isVIP: true)
    static let clark = Person(firstName: "Clark", lastName: "Kent", age: 120, isVIP: false /*I don't like superman, no VIP treatement for him*/))
}
```

We created a `struct` called `Person`. Next we created an `extension` of `Person` which conforms to `TableViewCellDataRepresentable`. Another `extension` just has some helper properties which we will be going use shortly.

That was all for our basic setup. The only new thing we need to do before we go into implementation of our ViewController is a `TableViewCellDecorator` which can decorate `ThreeLabelTableViewCell` using `Person`.

```swift
struct HorizontalDecorator: TableViewCellDecorator {
    static func update(tableViewCell cell: ThreeLabelTableViewCell, withData data: Person) {
        cell.stackView.layoutMargins = UIEdgeInsets(top: 0, left: cell.stackView.spacing, bottom: 0, right: cell.stackView.spacing)
        cell.stackView.isLayoutMarginsRelativeArrangement = true
        cell.toggle(VIPTheme: data.isVIP)
        cell.stackView.axis = .horizontal
        cell.firstLabel.text = data.firstName
        cell.secondLabel.text = data.lastName
        cell.thirdLabel.text = String(data.age)
    }
}
```

In above `TableViewCellDecorator`, the `CellType` and `DataType` are inferred from the paramter types of `update` function. So the complier has the concrete type for `CellType` as `ThreeLabelTableViewCell` and DataType as `Person`. Also there is no type casting involved as we won't be able to pass any other cell than `ThreeLabelTableViewCell` and any other dataType then `Person`.

Go through it once again, its not that complex.

If you are wondering about `toggle(VIPTheme:)` function, below is the snippet of all the helpers functions which I am using for this blog post.

```swift
extension ThreeLabelTableViewCell {
    
    func toggle(VIPTheme isVIP: Bool) {
        firstLabel.textColor = isVIP ? UIColor.VIP : UIColor.commonMan
        secondLabel.textColor = isVIP ? UIColor.VIP : UIColor.commonMan
        thirdLabel.textColor = isVIP ? UIColor.VIP : UIColor.commonMan
        firstLabel.font = isVIP ? UIFont.VIP : UIFont.commonMan
        secondLabel.font = isVIP ? UIFont.VIP : UIFont.commonMan
        thirdLabel.font = isVIP ? UIFont.VIP : UIFont.commonMan
    }
}

extension UIColor {
    static let VIP = UIColor.init(colorLiteralRed: 255/255, green: 215/255, blue: 0, alpha: 1)
    static let commonMan = UIColor.black
}

extension UIFont {
    static let VIP = UIFont.boldSystemFont(ofSize: 17)
    static let commonMan = UIFont.systemFont(ofSize: 17)
}

```

We will now see, that our `TableHandler` does not need to have any information about `Person` and `ThreeLabelTableViewCell` to display data from `Person` on `ThreeLabelTableViewCell`. Thats crazy enough for me to be fan of `protocol` and `associatedtype` in `Swift` 👍.

This also means that its very easy to hook up any other combination of decorator, cell and datatype with our `TableViewHandler`.

Now lets implement our ViewController.

```swift
class ViewController: UIViewController {
    @IBOutlet weak var tableView: UITableView!
    
    private var persons = [Person.vishal, Person.john]

    private lazy var tableViewDatasource: TableViewHandler<HorizontalDecorator> = {
        let tableHander = TableViewHandler<HorizontalDecorator>(withTableView: self.tableView, dataSet: self.persons, reuseIdentifiers: { _ in return "CellID" })
        return tableHander
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Persons"
        setUpTableView()
        // Do any additional setup after loading the view, typically from a nib.
    }
    
    private func setUpTableView() {
        tableView.register(UINib.init(nibName: "ThreeLabelTableViewCell", bundle: nil), forCellReuseIdentifier: "CellID")
        tableView.estimatedRowHeight = 44
        tableView.rowHeight = UITableViewAutomaticDimension
        tableView.dataSource = tableViewDatasource
    }
}
```

Thats it. Thats all in our ViewController. This is how it looks when we run it.

![Image alt](/assets/posts/Deep_Dive_Protocols/personTable.png "TableView")

Yea, not one of the most gorgeous user interface. Naming things is not the only thing in which I am **not** good at, but hey, its just a demo, so don't judge.

We can easily create a new decorator to change the look of our TableView. 

```swift
struct VerticalDecorator: TableViewCellDecorator {
    static func update(tableViewCell cell: ThreeLabelTableViewCell, withData data: Person) {
        cell.stackView.layoutMargins = UIEdgeInsets(top: cell.stackView.spacing, left: 0, bottom: cell.stackView.spacing, right: 0)
        cell.stackView.isLayoutMarginsRelativeArrangement = true
        cell.toggle(VIPTheme: data.isVIP)
        cell.stackView.axis = .vertical
        cell.firstLabel.text = data.firstName
        cell.secondLabel.text = data.lastName
        cell.thirdLabel.text = String(data.age)
    }
}
```

This is a `VerticalDecorator`, which will layout views of our `ThreeLabelTableViewCell` differently using same `Person` modal.

We just need to change the declaration of our `TableViewHandler` as below.

```swift
private lazy var tableViewDatasource: TableViewHandler<VerticalDecorator> = {
        let tableHander = TableViewHandler<VerticalDecorator>(withTableView: self.tableView, dataSet: self.persons, reuseIdentifiers: { _ in return "CellID" })
        return tableHander
    }()
```

Thats how our table looks now.

![Image alt](/assets/posts/Deep_Dive_Protocols/verticalLayout.png "VerticalLayout")

I know its looking even worse now.

But the idea is, you can use any combination of data type and cell with `TableViewHandler`. You can use another `TableViewCellDecorator` which decorates some other cell with `Person` data or which decorates `ThreeLabelTableViewCell` with some other data type or some other cell with some other data type. I hope you get my point 🙄.


## Extending TableViewHandler.
Apart from simply displaying data on table view, another most common operation we do is adding rows with new data. This is again something which has common logic but we write it everytime we have to do it.

```swift
//In TableViewHandler class, add this method.
func addData(data: [T.DataType], animated: Bool = true) {
        let startIndex = dataSet.count
        var indicesToAdd: [IndexPath] = []
        dataSet.append(contentsOf: data)
        if animated {
            for index in startIndex..<(data.count + startIndex) {
                indicesToAdd.append(IndexPath(row: index, section: 0))
            }
            tableView.beginUpdates()
            tableView.insertRows(at: indicesToAdd, with: .fade)
            tableView.endUpdates()
        } else {
            tableView.reloadData()
        }
    }
```

Its again data type independent. It will append new data and accordingly add new cells at the bottom of tableView.

While adding new cells was straight forward, deleting existing data and cells is little bit ticky. The implementation of removing data should somewhat look like below.

```swift
//In TableViewHandler class, add this method.

func removeData(data: [T.DataType], animated: Bool = true) {
        var indicesToRemove: [IndexPath] = []
        let proxyDataSet = dataSet
        for index in 0..<data.count {
            guard let _indexToRemove = proxyDataSet.index(where: { return $0 == data[index] }),
                let currentIndex = dataSet.index(where: { return $0 == data[index] })else {
                    continue
            }
            dataSet.remove(at: currentIndex)
            indicesToRemove.append(IndexPath(row: _indexToRemove, section: 0))
        }
        if animated {
            tableView.beginUpdates()
            tableView.deleteRows(at: indicesToRemove, with: .fade)
            tableView.endUpdates()
        } else {
            tableView.reloadData()
        }
    }


```

### Could you identify the problem?

The issue is, `==` operator. Compiler does not know how to perform equality check on `TableViewCellDataRepresentable` rightly because we haven't provided any information regarding this to compiler. Hence above code won't compile.

What can we do?

## Extension for rescue.

I will just go ahead and write the required code.

```swift
extension TableViewHandler where T.DataType: Equatable {
    
    func removeData(data: [T.DataType], animated: Bool = true) {
        var indicesToRemove: [IndexPath] = []
        for index in 0..<data.count {
            let indexToRemove = dataSet.index() { return $0 == data[index] }
            guard let _indexToRemove = indexToRemove else {
                continue
            }
            dataSet.remove(at: _indexToRemove)
            indicesToRemove.append(IndexPath(row: _indexToRemove, section: 0))
        }
        if animated {
            tableView.beginUpdates()
            tableView.deleteRows(at: indicesToRemove, with: .fade)
            tableView.endUpdates()
        } else {
            tableView.reloadData()
        }
    }
}
```

Now the `removeData` function is only avalibale if the `T.DataType`, i.e `TableViewCellDataRepresentable` conforms to `Equatable`. 

In our case, `Person` does not conform to `Equatable` so we wont be even able to call `removeData`. Thats mind blowing.

If we want `removeData` to be available for us, we can simply confrom our concrete `TableViewCellDataRepresentable` to `Equatable` like below.

```swift
extension Person: Equatable {
    static func == (lhs: Person, rhs: Person) -> Bool {
        return (lhs.firstName == rhs.firstName && lhs.lastName == rhs.lastName)
    }
}
```

Now we can call `removeData` as well.

## One last thing.

While adding new data, what if we want to perform duplicate check and just update the old version with new version if it already exist otherwise simply append it at bottom.

```swift
protocol Updatable: Equatable {
    mutating func update(fromCopy copy: Self)
}

// Still `Equatable` as `Updatable` inherits `Equatable`.
extension Person: Updatable {
    mutating func update(fromCopy copy: Person) {
        age = copy.age
        isVIP = copy.isVIP
    }
    static func == (lhs: Person, rhs: Person) -> Bool {
        return (lhs.firstName == rhs.firstName && lhs.lastName == rhs.lastName)
    }
}

extension TableViewHandler where T.DataType: Updatable {
   
    func addData(data: [T.DataType], updateExisting: Bool, animated: Bool = true) {
        var indicesToAdd: [IndexPath] = []
        var indicesToUpdate: [IndexPath] = []
        for index in 0..<data.count {
            let startIndex = dataSet.count
            let indexToUpdate = dataSet.index() { return $0 == data[index] }
            if let _indexToUpdate = indexToUpdate {
                dataSet[_indexToUpdate].update(fromCopy: data[index])
                indicesToUpdate.append(IndexPath(row: _indexToUpdate, section: 0))
                continue
            } else {
                indicesToAdd.append(IndexPath(row: startIndex, section: 0))
                dataSet.append(data[index])
            }
        }
        if animated {
            tableView.beginUpdates()
            tableView.insertRows(at: indicesToAdd, with: .fade)
            tableView.reloadRows(at: indicesToUpdate, with: .fade)
            tableView.endUpdates()
        } else {
            tableView.reloadData()
        }
    }
}
```

This is the basic setup I use for basic TableView operations.  

Thats it for this blog post. Let me know if you have any queries.

[Source Code](https://github.com/Swiftales/BasicTableViewDatasource)

Happy Swifting.🃏

