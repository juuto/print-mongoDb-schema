# print-mongoDb-schema
Query to print list of mongo db corrections with available fields and value examples

//print collection details: field name, type and value
function printSchema(collection, indent, collectionName) {
    for (var key in collection) {
        if (typeof collection[key] != "function" && typeof collection[key] != "object") {
            print(indent, 
            key + ": " + 
            typeof collection[key] + " | " + 
            collection[key] + "");
        } 
        var dateKeys = ["date", "initiatedOn", "validFrom", "valueDate", "openedOn", "changedOn"];
        if (typeof collection[key] == "object" && !dateKeys.includes(key)) {
            print(indent, key + "[]");
            if (collection[key].length < 10) {
                printSchema(collection[key], "--- ", collectionName);
            }
        }
        if (typeof collection[key] == "object" && dateKeys.includes(key)) {
            print(indent, key + ": date | ISODate(\'2018-07-14T00:00:00.000Z\')");
            if (collection[key].length < 10) {
                printSchema(collection[key], "--- ", collectionName);
            }
        }
        if (["status", "type", "paymentSystem", "transferType", "serviceType", "source", "provider", 
            "operationType", "registrationStatus", "partyType", "reasonCode", "paymentDetailsType", 
            "topUpServiceProvider", "plaisDocumentType", "plaisCardstaStatus", "plaisDocumentType", 
            "accountType", "schemeId"].includes(key)) {
            var listed = db.getCollection(collectionName).distinct(key);
            for (s = 0; s < listed.length; s++) {
                print("---", listed[s]);
            }
        }
    }
};

//get all collectiond
var allCollectionNames = db.getCollectionNames();

//call printSchema for each not empty collection
for (i = 0; i < allCollectionNames.length; i++) {
    if (db.getCollection(db.getCollectionNames()[i]).find({}).count() != 0) {   //if collection is not empty
        print(allCollectionNames[i]);        //print collenction name
        print("");
        // classes
        var allClasses = db.getCollection(db.getCollectionNames()[i]).distinct( "_class" );
        if (allClasses.length == 1) {     //collaction includes data from one class
            print("class: " + allClasses[0]);
            printSchema(db.getCollection(db.getCollectionNames()[i]).findOne({}), "-", allCollectionNames[i]);
        } else {        //collection includes data from several classes
           
            print("Collection data includes " + allClasses.length + " classes");
            for (var c in allClasses) {
                print("class: " + allClasses[c]);
                printSchema(db.getCollection(db.getCollectionNames()[i]).findOne({
                    "_class": allClasses[c]
                }), "-", allCollectionNames[i]);
            }
        }
        print("");
        print("");
    }
}
