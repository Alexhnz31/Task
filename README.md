Product2 prod = new Product2(
    Name = 'Test Product from Script',
    IsActive = true
);
insert prod;

Pricebook2 standardPB = [SELECT Id, IsActive FROM Pricebook2 WHERE IsStandard = TRUE LIMIT 1];

if (!standardPB.IsActive) {
    standardPB.IsActive = true;
    update standardPB;
}

PricebookEntry entry = new PricebookEntry(
    Pricebook2Id = standardPB.Id,
    Product2Id = prod.Id,
    UnitPrice = 123.45,
    IsActive = true,
    UseStandardPrice = false
);
insert entry;

Account acc = new Account(Name = 'Test Account');
insert acc;

Contact con = new Contact(
    FirstName = 'Lead',
    LastName = 'Convert',
    Email = 'lead.convert@example.com',
    AccountId = acc.Id
);
insert con;

Lead lead = new Lead(
    FirstName = 'Lead',
    LastName = 'Convert',
    Company = 'Test Company',
    Email = 'lead.convert@example.com',
    Status = 'Open - Not Contacted'
);
insert lead;

Database.LeadConvert lc = new Database.LeadConvert();
lc.setLeadId(lead.Id);
lc.setDoNotCreateOpportunity(false);
lc.setConvertedStatus('Closed - Converted');
Database.LeadConvertResult lcr = Database.convertLead(lc, true);

if (lcr.isSuccess()) {
    Opportunity opp = [SELECT Id FROM Opportunity WHERE Id = :lcr.getOpportunityId() LIMIT 1];

    OpportunityLineItem oli = new OpportunityLineItem(
        OpportunityId = opp.Id,
        Quantity = 1,
        PricebookEntryId = entry.Id,
        TotalPrice = 123.45
    );
    insert oli;
}
