---
Title: Making a Field Appear Required on a Visualforce Page
comments: true
---

I’ve been working on a force.com app with the requirement that a user must enter a valid email address on a Visualforce page before being able to save a record.

But they must also be able to insert the related contact’s email address by clicking a button instead of having to leave the edit page to go find it. That seemed simple enough, but it wasn’t. This is my solution for making a field appear required on a Visualforce page.

![Mail Folder](/assets/images/posts/visualforce-page.png#full "Apple Mail Folder")

In my original Visualforce page, it seemed logical that if I set the recipient email field as required, all would work as expected. However, defining the field as required prevented my custom action in my page controller from firing and entering the email address.

## Original Page Controller

````java
public class GiftCardTestController {
    private ApexPages.StandardController std;
    public String cEmail {get;set;}
    public Gift_Card_Order__c gc {get;set;}
    public GiftCardTestController(ApexPages.StandardController stdCtrl) {
        std = stdCtrl;
    }

    //selects the email address of the related contact
    //and inserts into recipient email field.
    public void fillEmail() {
        gc = (Gift_Card_Order__c)std.getRecord();
        cEmail = &#91;select Id, Email from Contact where Id = :gc.Contact__c].Email;
        gc.Recipient_Email__c = cEmail;
    }
}
````


## Original Visualforce Page

````html
<apex:page standardController="Gift_Card_Order__c" extensions="GiftCardTestController" title="Gift Card Test">
    <apex:form>
        <apex:pageblock title="Gift Card" mode="edit">
            <apex:pageblockbuttons location="top">
                <apex:commandbutton action="{!save}" value="Save"></apex:commandbutton>
                <apex:commandbutton action="{!cancel}" value="Cancel"></apex:commandbutton>
                <apex:commandbutton action="{!fillEmail}" value="Fill Email"></apex:commandbutton>
            </apex:pageblockbuttons>
            <apex:pageblocksection title="Email Info" columns="1">
                <apex:inputfield value="{!Gift_Card_Order__c.Contact__c}"></apex:inputfield>
                <apex:inputfield value="{!Gift_Card_Order__c.Recipient_Email__c}" required="true"></apex:inputfield>
            </apex:pageblocksection>
        </apex:pageblock>
    </apex:form>
</apex:page>
````


In this example, the ``fillEmail()`` action should select the related contact email address, and put the value in the ``Recipient_Email__c`` field so the user can see it.

But it’s not that simple it seems. When the field had the ``required="true"`` attribute set, the action would not fire because all validation is done on the client side and the page never posts back to the server — so the controller action never gets called.

So after some digging and asking for help on the Salesforce discussion boards, the solution was to make the ``Recipient_Email__c`` appear as if it’s required on the page (though it’s really not), and add a new save method to my controller to handle field validation on the server side when the record gets saved.

## New Page Controller

````java
public class GiftCardTestController {
    private ApexPages.StandardController std;
    public String cEmail {get;set;}
    public Gift_Card_Order__c gc {get;set;}
    public GiftCardTestController(ApexPages.StandardController stdCtrl) {
        std = stdCtrl;
    }

    public void fillEmail() {
        gc = (Gift_Card_Order__c)std.getRecord();
        cEmail = &#91;select Id, Email from Contact where Id = :gc.Contact__c].Email;
        gc.Recipient_Email__c = cEmail;
    }

    // add custom save method...
    public pageReference save() {
        gc = (Gift_Card_Order__c)std.getRecord();

        // if the recipient email is null, add an error to the field
        // and return null to remain on the current page...
        if(gc.Recipient_Email__c == null) {
            gc.Recipient_Email__c.addError('A valid email address is required.');
            return null;
        }

        // otherwise, the field is filled, so it's okay to redirect to view page.
        // standard field validation will check for valid email format.
        else {
            return std.save();
        }
    }
}
````

## New Visualforce Page

````html
<apex:page standardController="Gift_Card_Order__c" extensions="GiftCardTestController" title="Gift Card Test">
    <apex:form>
        <apex:pageblock title="Gift Card" mode="edit">
            <apex:pageblockbuttons location="top">
                <apex:commandbutton action="{!save}" value="Save"></apex:commandbutton>
                <apex:commandbutton action="{!cancel}" value="Cancel"></apex:commandbutton>
                <apex:commandbutton action="{!fillEmail}" value="Fill Email"></apex:commandbutton>
            </apex:pageblockbuttons>
            <apex:pageblocksection title="Email Information" columns="1">
                <apex:inputfield value="{!Gift_Card_Order__c.Contact__c}"></apex:inputfield>

                <!-- Updated pageBlockSectionItem -->
                <apex:pageblocksectionitem>
                    <apex:outputlabel>Email Recipient</apex:outputlabel>
                    <apex:outputpanel layout="block" styleClass="requiredInput">
                        <apex:outputpanel layout="block" styleClass="requiredBlock"></apex:outputpanel>
                        <apex:inputfield value="{!Gift_Card_Order__c.Recipient_Email__c}"></apex:inputfield>
                    </apex:outputpanel>
                </apex:pageblocksectionitem>
                <!--// end pageBlockSectionItem -->

            </apex:pageblocksection>
        </apex:pageblock>
    </apex:form>
</apex:page>
````

Notice the ``<apex:pageblocksectionitem />`` code to replace the original field. This is how we make the field appear with the “required” bar. A nifty trick that took some digging to discover. Hopefully, this post saves someone else the time it took me to figure it out — and me the time when I forget it.

For convenience, here’s a Github Gist with field label &amp; inline help that shows exactly how to make a field appear required on a Visualforce page.

<script src="https://gist.github.com/jimmitchell/4595878.js"></script>