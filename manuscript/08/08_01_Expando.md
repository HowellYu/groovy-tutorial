#Expando

I>The Expando class lets you create objects on the fly.

Before getting stuck into formal object definitions, let's take a look at the `Expando` class that ships with Groovy. Expando provides a very flexible way to construct an object on the fly.

In the example below the `monster` variable is dynamically assigned a set of fields and methods (through the use of closures).

{lang=Java}
<<[The Expando Monster](code/08/01/expando_monster.groovy)

The `monster` Expando looks to be acting in a manner similar to a Map but it gives us more functionality. Whilst the properties can be assigned to an Expando instance in the same way as a Map, the addition of a closure illustrates the difference. In the example below the use of a Map to define the `vampire` variable demonstrates that the `scareVillage` can't rely on an instance field (`name`) so must refer back to the `vampire` map:

{lang=Java}
<<[The Vampire Map](code/08/01/expando_vampire.groovy)

By using the `Expando` class instead of a `Map`, the instance fields can be accessed.

## Using Expando with CSV data

Expando can be useful when we want to consume data from a source (such as a file) and manipulate it as an object. One such example is an application that reads a comma-separated file (a CSV) that contains a header row and multiple subsequent data rows. The header row tells us the fields in our data objects and we can start to throw in some methods to help handle our data.

I'll start with a full listing of a script that accepts CSV data (I'll just use a String but it could be from a file) about various contacts. Expando will help me then produce a [vCard](https://en.wikipedia.org/wiki/VCard) for each contact. Take a read of the full listing and we'll then break it down into easier chunks.

{lang=Java}
<<[The full CSV example](code/08/01/expando_csv.groovy)

The handy `tokenize` method is used to break the CSV data into individual rows. The first row is actually a header row and we perform another `tokenize` to extract each header item (field) and then remove the header row from the data table:

    def csv = table.tokenize()
    header = csv[0].tokenize(',')
    csv.remove(0)

A `contactList` is then defined - this will hold our list of contacts:

    def contactList = []

The code now iterates through the data table.  For each row, an instance of `Expando` is declared:

    Expando contact = new Expando()

`tokenize` is again used to separate each field in the row. The `eachWithIndex` method passes the field's value and its index into the closure's key and value parameters respectively. I use the index to work out the field name in `header` and set the value as the field from the row:

    //Now setup the contact class with field names based on the header
    row.tokenize(',').eachWithIndex { key, value ->
        contact.setProperty header[value], key
    }

T> ## Another way to set the property
T> Instead of using `setProperty` I could have used Groovy's ability to use string interpolation to access a method/property:
T> 	`contact."${header[value]}" = key`

The `getVCard` closure is then added to the contact - this will construct a vCard for use in other systems:

    contact.getVCard = {
        """BEGIN:VCARD
        VERSION:4.0
        N:$name
        EMAIL:$email
        TEL;TYPE=work,voice;VALUE=uri:tel:$phone
        END:VCARD"""
    }
    contactList << contact


Once each row has been consumed we can display our `contactList`:

    for (contact in contactList) {
        println contact.getVCard()
    }
