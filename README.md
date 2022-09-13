# fsetgrp-api
FSET Groups (Groups) are a convenient way to associate user to a resources within the context of a group or team of individuals working on a project or projects within the FamilySearch umberalla.

Groups can be created by anyone and the creator is the original owner of the group. There is only one group owner but ownership can be transfered to another individual. Only the group owner can delete the group and ownership doesn't imply any other permissions to manage the group.

Group administration permissions is given by assignment and there may be multiple group administrators.

Groups do NOT represent or store permissions granted to any user. However, groups do allow members to request permissions using the assignments defined in the group. Requested assignments are not stored or managed as part of the group.

Adding users to a group does not imply permissions. A user can be a member of one or more groups without any assignments. The user may request any assignment of any group they are a member of.

Groups have an association to one or more resources. This association completes the resource-assignment-action paradigm that enables permission configuration in our IDP.

## Authentication and Authorization
### Authorization Header
```
"Authorization" : "Bearer \<token>"
```
---
## Common Error Handling
```
{
    "statusCode": <number>,
    "message": <string>,
    "uri" : <link>
}
```
### Details
- **statusCode** - Status code
- **message** - 
- **uri** - 

----
## FSET Resource Identifiers
The following FSET Resource Identifier (FRI) are used by this service:
- `fset:grp:group:<groupId>` -  FSET Group   
- `fset:grp:template:<templateId>` - FSET Template   
- `fset:grp:assignment:<user-type>:<role>:<environment>:<jurisdiction>:<level>` - FSET Assignment 

* * * 

## APIs
From here down are all the api calls for the service.

### Create Group
Creates a new group. When a new group is created, an FSETGrp Admin assignement is automatically added to the group for the Group owner. Additional FSETGrp Admin assignments may be added at a later time.

An error will be returned under the following conditions:
1. A group already exists with the same name
3. An invalid group owner specified. Likely due to non-existent user or wrong Organizational Role for the implied FSETGrp Admin assignement.
4. The group name does not meet the nameing requirements

```
create_group(GroupName='string', GroupDescription='string', Owner='string', Jurisdiction='string')
```
#### Parameters
- **GroupName** - Name of the group. This must consist of `[a-zA-Z][a-zA-Z0-9]{3,}`
- **GroupDescription** (optional) - Arbitrary description.
- **Owner** - User ID to be the group owner.
- **Jurisdiction** - Jurisdiction of group. Use `"Org"` for an organization. Don't specifiy for either group or self.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string',
    'Owner' : 'string',
    'IsOrgJurisdiction' : boolean
    }
}
```
#### Details
- **GroupName** (string) - Name of the group
- **GroupFri** (string) - Group FRI
- **Owner** - (string) - User ID of the group owner
- **IsOrgJurisdiction** (boolean) - `True` if the group is restricted to Organization jurisdiction assignments, `False` otherwise.

#### Enabling Assignments
- NONE REQUIRED


### List Groups
List all groups. If the caller has an assignment in a group, the results will always include those groups regardless of their assignments. Otherwise, an enabling assignment is necessary.

An error will be returned under the following conditions:
1. The caller does not have an enabling assignment to list groups AND has no assignments in any group.

```
list_groups(Filters=[{'Name' : 'string', 'Values' : ['string']}])
```

#### Parameters
- **Filters** (optional) - One or more filters
  - `Owner` - The ID of an owner to filter the results
  - `Assignments` - The FRI of one or more assignments to filter
  - `Users` - The FRI of one or more users

#### Response
```
{
    'Groups': {
        'GroupFri': 'string',
        'GroupName': 'string',
        'Owner': 'string',
        'IsOrgJurisdiction' : boolean
    }
}
```
#### Details
- **GroupName** - New name of the group
- **GroupFri** - FRI of the group
- **Owner** - User ID of the group owner
- **IsOrgJurisdiction** (boolean) - `True` if the group is restricted to Organization jurisdiction assignments, `False` otherwise.

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*` - Only needed to include groups the caller is not a member of.

### Describe Group
Get information about a single FSET Group.

An error will be returned under the following conditions:
1. The caller does not have necessary permissions to describe groups
2. The group does not exist

```
describe_group(GroupFri='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be described.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string',
    'Owner' : 'string',
    'IsOrgJurisdictions' : boolean,
    'AvailableAssignments': [
        'string',
    ],
    'Blueprints': [
        'string',
    ]
}
```
#### Details
- **GroupName** - Name of the group
- **GroupFri** - FRI of the group
- **Owner** - FRI of the owner
- **IsOrgJurisdiction** (boolean) - `True` if the group is restricted to Organization jurisdiction assignments, `False` otherwise.
- **AvailableAssignments** - List of available assignement FRIs associated to the group
- **Blueprints** - List of Blueprint FRIs associated to the group

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*`
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*`

### Delete Group
Deletes an existing group. 

An error will be returned under the following conditions:
1. The group does not exist
2. The caller is not the owner of the group. Only a group owner can delete a group.
3. Group has associated resources that require deleting first.

```
delete_group(GroupFri='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be deleted.

#### Response
```
{ }
```

### Update Group Attributes
Update group attributes. 

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have an enabling assignment to update group attributes
4. The group name does not meet the naming requirements

```
update_group_attributes(GroupFri='string', GroupName='string',
        GroupDescription='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **GroupName** (optional) - Name of the group. This must consist of `[a-zA-Z]([a-zA-Z0-9])+`
- **GroupDescription** (optional) - Arbitrary description.

#### Response
```
{ 'GroupAttributeInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string'
    }
}
```
#### Details
- **GroupName** - New name of the group
- **GroupFri** - FRI of the group

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*`
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*`

### Update Group Owner
Updates the group owner. When the owner is changed, an FSETGrp Admin assignment is automatically added for the new owner. The previous owner will retain the FSETGrp Admin assignment until it is explicitly removed.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have an enabling assignment to update the owner
3. An invalid owner is specified. Likely due to non-existent user or wrong Organizational role.

```
update_group_owner(GroupFri='string', Owner='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **Owner** - User ID of the new group owner.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string',
    'Owner' : 'string'
    }
}
```
#### Details
- **GroupName** (string) - Name of the group
- **GroupFri** (string) - Group FRI
- **Owner** - (string) - User ID of the group owner

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*`

### Add Assignments to a Group
Adds an assignment or set of assignments to the group. The assignments can then be used to make assignments to a user at a later time. If neither an assignment or template is specified, the operation silently does nothing.

If a template is specified, only the individual assignments contained in the template will be persisted on the group.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have an enabling assignment to add assignments
3. Invalid assignment due to mixed Jurisdictions
4. Invalid assignment due to Organization Jurisdiction restriction. Assignment with Organization jurisdiction can only be in one group.
5. Invalid template
6. Unknown assignment
7. Assignment already exists


```
add_assignment(GroupFri='string', AssignmentFri='string', TemplateName='string')
```
#### Parameters
- **GroupFri** - Id of the group to be updated.
- **AssignmentFri** (optional) - FRI of the rule being added to the group.
- **TemplateName** (optional)- Name of a template containing a set of assignments, all of which will be added. Duplicates will be silenty ignored.

#### Response
```
{ 'GroupRuleInfo' : {
    'GroupName' : 'string'
    'GroupFRI' : 'string'
    'Assignments' : [
        {'AssignmentFri' : 'string'}
    ]
}
```
#### Details
- **GroupName** - Name of the group
- **GroupFri** - FRI of the group
- **Assignments** - List of assignements assocated to the group
- **AssignmentFri** - FRI of an assignment

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:0-*:*`
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*`

### Remove Assignment from a Group
Remove an assignment from the group. Before an Assignment can be removed all users associated to the Assignement must be disassociated.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have an enabling assignment to remove the assignment
3. Assignment is not in the group
4. Assignment can not be removed from the group. This mainly applies to the FSETGrp Admin assignments. There must always be a FSETGrp Admin assigned.

```
remove_assignment(GroupFri='string', AssignmentFri='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **AssignmentFri** - FRI of an assignment

#### Response
```
{ 'GroupTemplateInfo' : {
    'GroupName' : 'string'
    'GroupFri' : 'string'
    'Assignments' : [
        {'AssignmentFri' : 'string'}
        ]
    }
}
```
#### Details
- **GroupName** - Name of the group
- **GroupFri** - FRI of the group
- **Assignments** - List of remaining assignments assocated to the group
- **AssignmentFri** - FRI of an assignment

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*`
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*`

### Create Assignment Template
Create an Assignment Template that can later be used to add rules to a group in an efficient manner.

An error will be returned under the following conditions:
1. The template already exists
2. The caller does not have an enabling assignment to create templates
3. The caller does not have an enabling assignment to read the group
4. Invalid assignment specified
5. Mixed Jurisdiction assignments not allowed

```
create_assignment_template(TemplateName='string', TemplateDescription ='string', Assignments=['string'], GroupFri='string')
```
#### Parameters
- **TemplateName** - Arbitrary name of the new template.
- **TemplateDescription** (optional) - Arbitrary description of the template
- **Assignments** (conditional) - List of individual assignment FRIs. A list of assignments OR a group FRI can be specified but not both.
- **GroupFri** (conditional)- A group FRI from which a template will be created.A list of assignments OR a group FRI can be specified but not both.

#### Response
```
{ 'TemplateInfo' : {
    'TemplateName' : 'string',
    'Assignments' : [
        {'AssignmentFri' : 'string'}
        ]
    }
}
```
#### Details
- **TemplateName** - Name of the template
- **Assignments** - List of assignments in the template
- **AssignmentFri** - FRI of a rule

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*`
- `fset:grp:assignment:*:fset-group-admin:*:g-*:*`
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` Required if a group was specified.
  

### Delete Assignment Template
Delete an Assignment Template.

An error will be returned under the following conditions:
1. The template does not exist
2. The caller does not have an enabling assignment to delete templates

```
delete_assignment_template(TemplateFri='string')
```
#### Parameters
- **TemplateFri** - FRI of the template to be deleted.

#### Response
```
{ }
```
#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*`
- `fset:grp:assignment:*:fset-group-admin:*:g-*:*`

### List Assignment Templates
List all Templates.

An error will be returned under the following conditions:
1. The caller does not have an enabling assignment to list templates

```
list_assignment_templates()
```
#### Response
```
{ 'Templates' : 
    [
        {
            'TemplateFri' : 'string',
            'TemplateName' : 'string',
            'TemplateDescription' : 'string'
        }
    ]
}
```
#### Details
- **TemplateFri** - FRI of the template
- **TemplateName** - Name of the template
- **TemplateDescription** - Description of the template

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:g-*:*` 


### Add Blueprint to a Group
Adds a Blueprint to the group. Before a Blueprint can be added to a group the Blueprint must exists in the Blueprint Meta Server and must have a valid owner.

The permissions to access the repository associated with a Blueprint may take up to 10 minutes to replicate into the FSET Identity provider.

An error will be returned under the following conditions:
1. The Group does not exist
1. The Blueprint does not exist
2. The Caller is not an owner of the Blueprint in the BP Meta Service
3. The caller does not have the enabling assignment to add resources to the group
4. Blueprint already assigned to a group.

```
add_blueprint_to_group(GroupFri='string', Blueprint='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **Blueprint** - The name of the blueprint being added to the group

#### Response
```
{ 'ResourceInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string',
    'Blueprint' : 'string'
    }
}
```
#### Details
- **GroupName** (string) - Name of the group
- **GroupFri** (string) - Group FRI
- **Blueprint** - (string) - Name of the Blueprint added

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` 



### Add AWS Full-Product Account to a Group
Adds an AWS full-product account to the group. A full-product account can be added to more than one FSETGrp.

An error will be returned under the following conditions:
1. The Group does not exist
2. The Full-Product account does not exist
3. The Caller does not have the enabling assignments to add product accounts to the group. Please notice that both assignments listed in the `Required Assignments` section are required to enable this API.

```
add_product_account_to_group(GroupFri='string', ProductAccount='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **ProductAccount** - The 12-digit AWS AccountId.

#### Response
```
{ 'ResourceInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string',
    'AccountId' : 'string'
    }
}
```
#### Details
- **GroupName** (string) - Name of the group
- **GroupFri** (string) - Group FRI
- **AccountId** - (string) - 12-digit AWS AccountId of the added account.

#### Required Assignments
Both of the following assignments are required:
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` 
- `fset:grp:assignment:*:product-account-owner:*:o-*:ExpertContributor`- May need a new level for this.



### List Resources in a Group
List all resources associated to the group. 

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have an enabling assignment to list resources

```
list_resources(GroupFri='string')
```
#### Parameters
- **GroupFri** - FRI of the group for which repos will be listed.

#### Response
```
{ 
    'Resources' : {
        'GroupName' : 'string',
        'GroupFri' : 'string',
        'PrimitiveAccount' : 'string',
        'Blueprints' : [
            {
                'BlueprintName' : 'string'
            }
        ],
        'ProductAccounts' : [
            {
                'AccountId' : 'string'
            }
        ]
    }
}
```
#### Details
- **GroupFri** - FRI of the FSET Group
- **GroupName** - Name of the FSET Group
- **PrimitiveAccount** - The 12-digit AWS account Id for the primitive account associated to the group. This will not be present if there is no such association.
- **Blueprints** - List of the blueprints
  - **BlueprintName** - Blueprint name
- **ProductAccounts** - List of the full product accounts
  - **AccountId** - The 12-digit AWS account id

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*`
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*`
- `fset:grp:assignment:*:product-account-owner:*:o-*:*`


### Remove a Blueprint from a Group
Remove a Blueprint from the group. 

If the Blueprint has an associated source repository, permissions to the repository will be removed, including read access. The removal of permissions to the repository may take up to 10 minutes to be replicated into the FSET Identity provider.

An error will be returned under the following conditions:
1. The group does not exist
2. The blueprint isn't associated to the group
2. The Caller is not an owner of the Blueprint in the BP Meta Service
3. The caller does not have an enabling assignment to remove resources

```
remove_blueprint_from_group(GroupFri='string', Blueprint='string')
```
#### Parameters
- **GroupFri** - FRI of the group for which the repo will be removed.
- **RepoUrl** - The name of the blueprint

#### Response
```
{ }
```
#### Details
No response details

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*` 
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` 

### Remove an AWS Full-Product Account from a Group
Removes an AWS full-product account from the group. A full-product account must be associated to at least one full-product account.

An error will be returned under the following conditions:
1. The Group does not exist
2. The Full-Product is not associated to the group
3. The Full-Product account can't be removed without being orphaned (not assoicated to another account).
4. The Caller does not have the enabling assignments to remove product accounts from the group. Please notice that both assignments listed in the `Required Assignments` section are required to enable this API.

```
remove_product_account_to_group(GroupFri='string', ProductAccount='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **ProductAccount** - The 12-digit AWS AccountId.

#### Response
```
{}
```
No result details

#### Required Assignments
Both of the following assignments are required:
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` _**OR**_ `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*`
- `fset:grp:assignment:*:product-account-owner:*:o-*:ExpertContributor` - May need a new level for this.


### Add User to a Group
Add a user to a group and request the provisioning of one or more assignments for the user. The user may already be part of the group in which case only assignmens will provisioned. Assignment provisioning is idempotent and will be regenerated.

The assignment provisioning is asyncrounous and may be denied. In this case, the API return successfully without any indication of the provisioning process.

An error will be returned under the following conditions:
1. Invalid group specified
2. Invalid user specified
3. The caller does not have an enabling assignment to add users
4. One or more specified assignments are not present in the group

```
add_user_to_group(GroupFri='string', UserFri='string', Assignments=['string'])
```
#### Parameters
- **GroupFri** - FRI of the group to which the user will be added
- **UserFri** - FRI of the user being added
- **Assignments** - List of individual assignment FRIs.

#### Response
```
{}
```
#### Details
No Details returned

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*` 
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` 

### Request Assignment in a Group
Request on or more assignments be provisioned for a user. If the caller is a member of the group, the user can request an assignment be provisioned without specific permissions.
If the caller is not a member of the group the caller must have an enabling assignment.

The assignment provisioning is asyncrounous and may be denied. In this case, the API return successfully without any indication of the provisioning process.

An error will be returned under the following conditions:
1. Invalid group specified
2. Invalid user specified
3. One or more specified assignments are not present in the group

```
request_assignment(GroupFri='string', UserFri='string', Assignments=['string'])
```
#### Parameters
- **GroupFri** - FRI of the group to which the user will be added
- **UserFri** - FRI of the user being added
- **Assignments** - List of individual assignment FRIs.

#### Response
```
{}
```
#### Details
No Details returned

#### Enabling Assignments
These are required when the caller is different than the user specified in the API
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*` 
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` 
