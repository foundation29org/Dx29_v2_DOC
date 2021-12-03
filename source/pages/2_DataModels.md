<img align="right" width="100px" src="../_images/Foundation29.png">

# 2. Data models

A Dx29 user is the element that contains the credential information and access permissions to the platform.

This element will be:
>- On the one hand, the one in charge of managing access to the application.
>- On the other hand, the point of differentiation between the different profiles of the application, that is, it will be in charge of restricting the permissions and access to the data and its management or to the different parts of the application.

Thus, the information or data of a user of the platform will be mainly:
>- Credentials: user name, email and password.
>- User group: Here a distinction will be made between patients and doctors.
>- Role: User or administrator

The difference between patient and doctor is:
>- A patient user will only be able to manage their medical history.
>- A doctor user can create several medical records, without the need for a patient user to exist.

We are therefore talking about the management of medical records by each profile.

A medical history is subdivided into two elements:
>- Medical History Case (MHC), which is stored in the "caseRecords" database and contains the patient's personal information.
>- Medical History (MH), which will be the medical records associated with a patient, stored in the "medicalCases" database and containing the patient's medical information.

In turn, a Medical History will be made up of different resources, which can be grouped by type: symptoms, genes, medical reports, related diseases, etc. In other words, only the pointers to the groups of resources in the medical history will be stored in this database. These groups will be stored in a third database: resourceGroups.

It is important to take into account that, due to the definition of doctor and patient:
>- A patient user will have:
>>- His own medical history, which is the one he has created when registering in the application and filling in his data.
>>- He/she may have other associated medical histories if he/she has shared his/her case with a doctor. That is, when a patient's case is shared with a doctor, the doctor will create a new medical history related to that of the patient with whom he/she will work and that the patient will be able to see. In this way it is possible to know who has made the changes to a medical case, even though the user is presented with all the information and this is transparent to him.
>- A medical user will have:
>>- The medical histories that he/she has created without associating them to any patient in the Dx29 application.
>>- The medical histories shared by the patients of the Dx29 application.
>>- You will be able to receive medical histories also from other doctors. In the case that a doctor shares with another doctor, the same logic occurs as between patient and doctor, that is, a new medical history will be created for the second doctor, although this will be transparent to the users.

The following subsections will define the structure of the objects that are stored in each of the above-mentioned databases.

## 2.1. Medical History Case (MHC) - CaseRecords

```
{
    "id": <mhc_id>,
    "userId": <user_id_crypt>,
    "type": <user_type>,
    "properties": {
        "name": <patient_name>,
        "birthDate": <patient_birthdate>,
        "gender": <patient_gender>
    }
}
```
Each Medical history case will contain:
>- An identifier or id.
>- The encrypted identifier of the SQL database that contains the credentials of the users registered in the application.
>- The type of the user: Patient or Physician.
>- An object with the properties of the Medical History Case: The name, date of birth and gender of the patient.

## 2.2. Medical History (MH) - MedicalCases

```
{
    "id": <mh_id>,
    "userId":  <user_id_crypt>,
    "status": <mh_status>,
    "resourceGroups": {
        <resource_group_id>: {
            "type": <resource_group_type>,
            "name": <resource_group_name>
            "resources": {
                <resource_id>: <resource_status>
            },
            "lastUpdate": <last_resource_update_date>
        }
    },
    "sharedWith": [
    	{
            "userId": <user_id_who_sharedWith>,
            "status": <sharing_with_status>,
            "lastUpdate": <last_sharingAction_update_date>
        }
    ],
    "sharedBy": [
    	{	
            "userId": <user_id_who_sharedBy>,
            "caseId": <main_mh>,
            "status": <sharing_by_status>,
            "lastUpdate": <last_sharingAction_update_date>
        }
     ],
    "createdOn": <mh_createdOn_date>,
    "updatedOn": <last_mh_update_date>,
    ```
    ```
}
```
Each Medical history case will contain:
>- An identifier or id. If the case is a shared case this identifier will start with "s", while if it is an owner's case or private it will start with "c".
>- The encrypted identifier of the SQL database that contains the credentials of the users registered in the application.
>- The status of the case: Deleted, Shared, Private, ...
>- The creation and update dates of the case.
>- A list of the resource groups associated with the case. Each resource group will be an object in this list with:
>- Key equal to the resource group identifier.
>- Value, an object with:
>>- The type of resource group: Phenotype, Gene, Report,....
>>- The name associated to this resource group. This is used to differentiate for example between manual symptoms and symptoms extracted from a report.
>>- The creation and update dates of the resource group.
>- If applicable, a list of users with whom the case has been shared (shared with):
>>- The identifier of the user with whom the case has been shared.
>>- The status of the sharing: Pending approval, shared, rejected or revoked.
>>- The date of update of the actions on the shared case.
>- If applicable, a list of the users who have shared the case with me (SharedBy):
>>- The identifier of the user who has shared the case with me.
>>- The case identifier of the user who owns the case
>>- The status of the sharing: Shared, pending, rejected or revoked.
>>- The date of update of the actions on the sharing of the case.

**List of posible Medical History status:**
>- Private: The case has been created by a user and is not shared by the user
>- Shared: The case was shared with me by another user
>- Sharing: The case is shared with another user
>- PendingShared: The sharing of the case is pending approval by the case owner.
>- StopShared: The case is no longer shared
>- Deleted: The case has been deleted

**List of posible sharing status:**
>- If the case is shared from owner to another user (SharedWith):
>>- Shared: The case was shared with me by another user
>>- Pending: The sharing of the case is pending approval by the case owner.
>>- Revoked: The sharing of this case is revoked.
>- If the case is shared with this user form another user (SharedBy):
>- Pending: The sharing of the case is pending approval by the case owner.
>- Accepted: The case owner has accepted the sharing.
>- Rejected: The case owner has rejected the sharing.
>- Revoked: The case owner has revoked the sharing.

**List of resource group types**
>- Reports: For medical reports of the medical case.
>- Phenotype: For phenotype or symptoms of the medical case.
>- Genotype: For genotype reports of the medical case.
>- Analysis: Suggested diseases results of Dx29 algorithm on the patient case
>- Notes: Notes of the medical case
>- TimeLine: Symptoms chronology data of the medical case.

**List of resource group names**
>- Medical: Medical report.
>- Genetic: Genotype report.
>- Manual: Phenotype or Genotype selected by the user.
>- Analysis: Suggested diseases results of Dx29 algorithm on the patient case
>- Notes: Notes of the medical case

## 2.3. Resources grouped by type - ResourceGroups

```
{
    "id": <resource_group_id>,
    "userId": <user_id_crypt>,
    "caseId": <mh_id>,
    "type": <resource_group_type>,
    "name":  <resource_group_name>,
    "resources": {
        <resource_id>: {
            "id": <resource_id>,
            "name": <resource_name>",
            "status": <resource_status>,
            "properties": <resource_properties>,
            "createdOn": <resource_createdOn_date>,
            "updatedOn": <resource_updatedOn_date>
        }
    },
    "createdOn": <resource_group_createdOn_date>,
    "updatedOn": <resource_group_updatedOn_date>,
}
```
Each resource group will contain:
>- An identifier or id.
>- The encrypted identifier of the SQL database that contains the credentials of the users registered in the application.
>- The creation and update dates of the resource group.
>- The type of resource: Phenotype, Gene, Report,....
>- The name associated to this resource group. This is used to differentiate for example between manual symptoms and symptoms extracted from a report.

>- A list of the resources associated with the resource group. Each resource will be an object in this list with:
>- Key equal to the resource identifier.
>- Value, an object with:
>>- The identifier of the resource
>>- The name of the resource: the file name, the symptom name, etc. which will be display on Dx29 application.
>>- The status of the resource: undefined, selected, unselected
>>- The creation and update dates of the resource.
>>- An object with the properties of the resource: It is a dictionary with string as key and string as value.  This will be different for each type or resource:
>>>- For symptom: no properties.
>>>- For gene: ```Properties = {"Score":<string_Score_value>, "Diseases":<string_diseasesList>}```
>>>- For report: ```Properties = {"Size":<fileSize_string>}```
>>>- Analysis: 
```
Properties = {
    "Symptoms":<symptomsList_string>, 
    "Genes":<genesList_string>, 
    "GenotypeIds":<genotypeReportsList_string>, 
    "Diseases":<diseasesList>_string>
}
```
>>>- Notes: no properties.
