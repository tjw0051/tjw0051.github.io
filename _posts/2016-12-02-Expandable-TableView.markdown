---
layout:     post
title:      "iOS Expandable Table View Cells"
subtitle:   "Expanding TableView Cells via Cell insertion and deletion."
date:       2016-12-02 12:30:00
author:     "Tom"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - iOS
    - Swift
---

# Overview

The following is an implementation for creating an expanding TableView cell effect by insertion and deletion of cells.

There's many ways to create the appearance of expanding TableView cells, most obviously by increasing the cell's height to expose overflow contents of the cell. After checking out other methods I decided to go with cell-insertion for my implementation for several reasons:
1. The cell 'expansion' has a different appearance to the default cell.

2. Its easier to insert a cell than to stretch an existing cell while maintaining the desired layout constraints.

3. The Expanded cell can be designed separately from the default cell, and dynamically added as an expansion to any cell in the TableView.

4. This method doesn't explicitly define the state of the cell as 'expanded' or 'contracted'. Little/no additional data needs to be stored for the expanded cells.

If you're implementing this in an existing project, [skip the first section](#section_skip).

If you'd like to go straight to the finished project, [click here](#section_download).

---

# Setting Up

Open up xCode and create a new Xcode Project. On the project template window, select 'Single View Application' and click Next.

![Screenshot](http://i.imgur.com/kIEPjTD.png)

On the project options screen, fill out the 'Product Name', 'Organisation Name', and 'Organisation Identifier' shown below:

![Screenshot](http://imgur.com/s6t9PrP.png)

When the project has been created, open the Main.storyboard file from the Project Navigator. You will be presented with a single blank ViewController.

![Screenshot](http://imgur.com/QM8tsAT.png)

 Delete it and drag a new TableViewController into the storyboard from the Object Library.

![Screenshot](http://imgur.com/nKKXhUz.png)

Select the newly created TableViewController from the hierarchy. In the Attributes Inspector, under the 'View Controller' category check the 'Is Initial View Controller' box.

![Screenshot](http://imgur.com/bjh8zmL.png)

Select the TableView inside the TableViewController from the hierarchy. In the attributes inspector under the 'TableView' category increase the 'Prototype Cells' property to 2. This will allow us to design the default cell and the expansion cell within the the same TableView.

![Screenshot](http://imgur.com/8e2dckt.png)

<p id = "section_skip"></p>

Drag 3 new Swift files from the File Template Library into the project folder.

![Screenshot](http://imgur.com/x2t3AjJ.png)

 Name the 3 files 'DefaultCell.swift', 'ExpansionCell.swift' and 'ExpandingTVC.swift' as follows:

![Screenshot](http://imgur.com/HDmpVka.png)

 Open the DefaultCell.swift file and enter the following:

```swift
import Foundation
import UIKit

class DefaultCell : UITableViewCell {

}
```

Next, open the ExpansionCell.swift file and enter:

```swift
import Foundation
import UIKit

class ExpansionCell : UITableViewCell {

}
```

Finally, in the ExpandingTVC.swift file, enter:

```swift
import Foundation
import UIKit

class ExpandingTVC : UITableViewController {

}
```

This will allow us to link the classes to the views in the storyboard. We'll write the full implementation for these later.

Open up the Main.storyboard again and select the first cell in the TableView. In the Identity Inspector, change the custom class to 'DefaultCell':

![Screenshot](http://imgur.com/We37dQ2.png)

Do the same for the second cell:

![Screenshot](http://imgur.com/I7RfE1l.png)

And for the TableViewController:

![Screenshot](http://imgur.com/3TuBKPk.png)

Select the Default cell again and change the cell height to 142:

![Screenshot](http://imgur.com/pC3BELG.png)

On the attributes inspector, change the Identifier name to 'DefaultCell', and change the background color to black (this will make it easier to see the white text).

![Screenshot](http://imgur.com/W2bfJIq.png)

![Screenshot](http://imgur.com/fjZaHsm.png)

Do the same for the ExpansionCell, setting its Identifier name to 'ExpansionCell'. Leave the background color of this cell white.

# Creating the Table Cells

Drag two Text Labels onto the DefaultCell and arrange them roughly as follows:

![Screenshot](http://imgur.com/WluFRTl.png)

The 'San Francisco' label is white, with size 40 and style 'light'. The '£425' label is white, with size 27 and style 'Black'.

Open the Assistant Editor to view the code next to the storyboard.

![Screenshot](http://imgur.com/KVLlNV0.png)

If the class in the AssistantEditor is not the DefaultCell class, you may need to select it manually, like this:

![Screenshot](http://imgur.com/DDcGDyV.png)

Select the San Francisco Text Label. Holding CTRL, click-and-drag from the Text Label to the class inside the assistant editor. In the popup dialog, set the name of the outlet to 'DestinationLabel'.

![Screenshot](http://imgur.com/DpZd5ua.png)

Do the same for the '£425' label, naming the outlet to 'PriceLabel'. Your DefaultCell.swift file should now look like this:

```swift
class DefaultCell : UITableViewCell {
    @IBOutlet weak var DestinationLabel: UILabel!
    @IBOutlet weak var PriceLabel: UILabel!

}
```

Moving on to the ExpansionCell, drag a new ImageView component from the object library and position it inside the ExpansionCell.

![Screenshot](http://imgur.com/docONIO.png)

 On the ImageView attributes change the image to the flight-icon imported earlier.

![Screenshot](http://imgur.com/ujiZ9ry.png)

 Add two Labels to the ExpansionCell with size 32, positioned approximately like this:

![Screenshot](http://imgur.com/CcjHChg.png)

Now things are starting to take shape!

Again we need to link the two labels 'MAN' and 'CFO' to the corresponding class. Select the ExpansionCell from the hierarchy and open the Assistant Editor. Make sure that the ExpansionCell class is open. As with the DefaultCell, hold CTRL and click-and-drag from the label in the storyboard to the class file. The outlet for label 'MAN' is StartLabel, and the outlet for label 'CFO' is EndLabel. Your `ExpansionCell` class should now look like this:

```swift
class ExpansionCell : UITableViewCell {
    @IBOutlet weak var StartLabel: UILabel!
    @IBOutlet weak var EndLabel: UILabel!
}
```

## Defining the Data

Next, we need to lay a bit of ground-work for our data. We need to store information for each destination shown in the `TableView`. Each destination needs a:
* Name
* Price
* Image Name
* List of Flights

Each flight in the list needs a start point and an end point. To set this up, create a new Swift file in the project called DestinationData.swift. Insert the following code into the file:

```swift
import Foundation

public class DestinationData {

    public var name: String
    public var price: String
    public var imageName: String
    public var flights: [FlightData]?

    init(name: String, price: String, imageName: String, flights: [FlightData]?) {
        self.name = name
        self.price = price
        self.imageName = imageName
        self.flights = flights
    }
}

public class FlightData {
    public var start: String
    public var end: String

    init(start: String, end: String) {
        self.start = start
        self.end = end
    }
}
```

As a brief explanation of whats happening here, each destination is stored in an instance of the `DestinationData` class. Each flight, represented by the ExpansionCell, is stored in the `flightData` class. Each destination stores multiple flights in an array. The array of flights is optional, denoted by the ?. This is to demonstrate the case where some cells will not expand (as they have no flights available).

Now we can move on to setting up the `TableViewController`.

# Setting up the Table

Open the `ExpandingTVC` class and create the following function for generating some destination data:

```swift
private func getData() -> [DestinationData?] {
        var data: [DestinationData?] = []

        let sanFranciscoFlights = [FlightData(start: "MAN", end: "CFO")]
        let sanFrancisco = DestinationData(name: "San Francisco", price: "£425", imageName: "san_francisco-banner", flights: sanFranciscoFlights)

        let londonFlights = [FlightData(start: "MAN", end: "LHR"), FlightData(start: "MAN", end: "LCY")]
        let london = DestinationData(name: "London", price: "£500", imageName: "london-banner", flights: londonFlights)

        let newYork = DestinationData(name: "New York", price: "630", imageName: "new_york-banner", flights: nil)

        return [sanFrancisco, london, newYork]
    }
```
Here we create 3 destinations: San Francisco, London and New York. San Francisco has 1 flight, London has 2, and New York has 0, and so will not expand.

Before the newly-created `getData()` function, add a variable `DestinationData` and override the `viewDidLoad` function, as follows:

```swift
var destinationData: [DestinationData?]?

    override func viewDidLoad() {
        destinationData = getData()
        tableView.rowHeight = UITableViewAutomaticDimension;
    }
```
This will store the `destinationData` in the `TableViewController` and allow the `TableView` rows to automatically set their height. Note that both the `destinationData` array, and the contents of the array are optional. This is to allow us to insert `nil` values into the array, to represent the location of expanded cells in the TableView.

Next, override the `tableView` function for specifying the number of rows, as shown below:
```swift
/*  Number of Rows  */
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        if let data = destinationData {
          return data.count
        } else {
          return 0
        }
    }
```

## Displaying Cells

To tell the `TableView` which cell to use, and to populate it with data, we next override the following function:
```swift
/*  Create Cells    */
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    }
```

To track where we put expanded cells in the `TableView`, we're going to insert `nil` entries into the `destinationData` array. So first we need to check if the cell being created is a `DefaultCell` or an `ExpansionCell`. This is easily accomplished by checking if the index in the array is nil or not:

```swift
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    // Row is DefaultCell
    if let rowData = data[indexPath.row] {
      let defaultCell = tableView.dequeueReusableCell(withIdentifier: "DefaultCell", for: indexPath) as! DefaultCell
                  defaultCell.DestinationLabel.text = rowData.name
                  defaultCell.PriceLabel.text = rowData.price
                  defaultCell.backgroundView = UIImageView(image: UIImage(named: rowData.imageName))
                  defaultCell.selectionStyle = .none
                  return defaultCell
    }
    // Row is ExpansionCell
    else {

    }
}
```

If the row is a `DefaultCell`, the cell is dequeued using the 'DefaultCell' identifier, and the destination name, price, and background image are set.

Next we need to set the data for an expansion cell. As mentioned earlier, an expansion cell is represented by a `nil` entry in the `destinationData` array, therefore we need to find which data is associated with the `nil` entry. One option is to get the cell before the expansion, and use its data to populate the `ExpansionCell`. This solution works for 1 `ExpansionCell`, but if there are multiple `ExpansionCell`s, then the cell above the current cell may also be nil:

![Screenshot](http://imgur.com/iqiA6IZ.png)

To get the 'parent' cell we need to go up the chain of cells until we find one that is not nil. This is accomplished by the following function. Put this function in the `ExpandingTVC` class to find the parent cell:

```swift
/*  Get parent cell for selected ExpansionCell  */
    private func getParentCellIndex(expansionIndex: Int) -> Int {

        var selectedCell: DestinationData?
        var selectedCellIndex = expansionIndex

        while(selectedCell == nil && selectedCellIndex >= 0) {
            selectedCellIndex -= 1
            selectedCell = destinationData?[selectedCellIndex]
        }

        return selectedCellIndex
    }
```

With this utility method, we can go ahead and implement the creation of `ExpansionCell`s. Back in the `tableView(... cellForRowAt ...)` function, insert the following inside the else parentheses:

```swift
// Row is ExpansionCell
else {
    if let rowData = data[getParentCellIndex(expansionIndex: indexPath.row)] {
        //  Create an ExpansionCell
        let expansionCell = tableView.dequeueReusableCell(withIdentifier: "ExpansionCell", for: indexPath) as! ExpansionCell

        //  Get the index of the parent Cell (containing the data)
        let parentCellIndex = getParentCellIndex(expansionIndex: indexPath.row)

        //  Get the index of the flight data (e.g. if there are multiple ExpansionCells
        let flightIndex = indexPath.row - parentCellIndex - 1

        //  Set the cell's data
        expansionCell.StartLabel.text = rowData.flights?[flightIndex].start
        expansionCell.EndLabel.text = rowData.flights?[flightIndex].end
        expansionCell.selectionStyle = .none
        return expansionCell
    }

}
```

First we dequeue an `ExpansionCell` using the identifier defined earlier. We then get the index of the parent cell using the `getParentCellIndex()` function. Next, we need to get the index of the flight data that will be displayed in THIS `ExpansionCell`. We get this from subtracting the index of THIS `ExpansionCell` from the index of the parent `DefaultCell`. We subtract a further 1 to ensure the index starts at 0. Finally we set the cell's data and return it.

Your tableView(... cellForRowAt ...) method should now look like this:
```swift
/*  Create Cells    */
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // Row is DefaultCell
        if let rowData = destinationData?[indexPath.row] {
            let defaultCell = tableView.dequeueReusableCell(withIdentifier: "DefaultCell", for: indexPath) as! DefaultCell
            defaultCell.DestinationLabel.text = rowData.name
            defaultCell.PriceLabel.text = rowData.price
            defaultCell.backgroundView = UIImageView(image: UIImage(named: rowData.imageName))

            return defaultCell
        }
        // Row is ExpansionCell
        else {
            if let rowData = destinationData?[getParentCellIndex(expansionIndex: indexPath.row)] {
                //  Create an ExpansionCell
                let expansionCell = tableView.dequeueReusableCell(withIdentifier: "ExpansionCell", for: indexPath) as! ExpansionCell

                //  Get the index of the parent Cell (containing the data)
                let parentCellIndex = getParentCellIndex(expansionIndex: indexPath.row)

                //  Get the index of the flight data (e.g. if there are multiple ExpansionCells
                let flightIndex = indexPath.row - parentCellIndex - 1

                //  Set the cell's data
                expansionCell.StartLabel.text = rowData.flights?[flightIndex].start
                expansionCell.EndLabel.text = rowData.flights?[flightIndex].end

                return expansionCell
            }
        }
        return UITableViewCell()
    }
```

We also need to tell the `TableView` what height to use for the different types of cells. The following function does this:

```swift
override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        if let rowData = destinationData?[indexPath.row] {
            return 142
        } else {
            return 75
        }
    }
```

# Expanding and Contracting Cells

To make the code a bit clearer, we're going to split up the logic for expanding and contracting cells. This will require 2 function: `expandCell()` and `contractCell()`.

For the expanding the cell, create the following function:

```swift
private func expandCell(tableView: UITableView, index: Int) {
        // Expand Cell (add ExpansionCells
        if let flights = destinationData?[index]?.flights {
            for i in 1...flights.count {
                destinationData?.insert(nil, at: index + i)
                tableView.insertRows(at: [NSIndexPath(row: index + i, section: 0) as IndexPath] , with: .top)
            }
        }
    }
```

In the above code, we first check if the cell to be expanded has any flights stored. If it does, the for-loop proceeds through each flight, inserting nil into the `destinationData` array (to represent the location of an `ExpansionCell`), and calls the `tableView.InsertRows()` function. In the `insertRows()` function we pass the index where we want to insert a row, and the animation to use. We're using the `.top` animation for this, as it 'pull-down' the new cell from the one above it, giving the illusion of the cell expanding from under the above cell.

For contracting a cell, insert the following function into the `ExpandingTVC` class:

```swift
private func contractCell(tableView: UITableView, index: Int) {
        if let flights = destinationData?[index]?.flights {
            for i in 1...flights.count {
                destinationData?.remove(at: index+1)
                tableView.deleteRows(at: [NSIndexPath(row: index+1, section: 0) as IndexPath], with: .top)

            }
        }
    }
```

this function works similarly to the `expandCell()` function, but instead deletes entries from the `destinationData` array and removes rows from the table. This also uses the `.top` animation to show the removed cell sliding up under the above cell.

# Selecting Cells

Finally we connect our logic to the `tableView(... didSelectCell...)` method, for when a user clicks on a cell. First we define the function and then check if the user clicked on a `DefaultCell`. (For the purposes of this tutorial, we're only expanding/contracting cells when the user clicks on the 'parent' `DefaultCell` - nothing happens when the user clicks on an ExpansionCell):

```swift
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        if let data = destinationData?[indexPath.row] {
```

The next bit gets a bit complicated. We need to see if the cell below the selected cell is `nil` or not, to check if the current cell is expanded. If the current cell is the last cell in the array, when we try to access the cell at indexPath.row + 1, we'll get an 'Array Index out of Range' error. So first we check if this is the last cell in the array:

```swift
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        if let data = destinationData?[indexPath.row] {

            // If user clicked last cell, do not try to access cell+1 (out of range)
            if(indexPath.row + 1 >= (destinationData?.count)!) {
                expandCell(tableView: tableView, index: indexPath.row)
            }
```

If the user DOES click the last cell, then the only option is to expand it, so we call the `expandCell()` function defined earlier. Now that this dodgy edge-case is taken care of, we can easily check the next cell below the current one:

```swift
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        if let data = destinationData?[indexPath.row] {

            // If user clicked last cell, do not try to access cell+1 (out of range)
            if(indexPath.row + 1 >= (destinationData?.count)!) {
                expandCell(tableView: tableView, index: indexPath.row)
            }
            else {
                // If next cell is not nil, then cell is not expanded
                if(destinationData?[indexPath.row+1] != nil) {
                    expandCell(tableView: tableView, index: indexPath.row)
                // Close Cell (remove ExpansionCells)
                } else {
                    contractCell(tableView: tableView, index: indexPath.row)

                }
            }
        }
    }
```

And thats it! Run the code and you should now be able to expand cells 1 and 2.

---

# Download - Github

<p id = "section_download"></p>

The full source code for the project can be found on Github at:

[https://github.com/codebasecampprojects/Expanding-TableView](https://github.com/codebasecampprojects/Expanding-TableView/tree/master)


—— Tom 2015.11
