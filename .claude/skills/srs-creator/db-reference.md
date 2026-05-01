## Database Schema

### entity
A person or company

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| entity_id | UUID | PK | | | Y |
| linkedin_url | Text | | | | |
| website_url | Text | | | | |
| location | Text | | | | |
| notes | Text | | | | |
| created | Date | | | | Y |
| last_updated | Date | | | | Y |

### person
A person

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| person_id | UUID | PK, FK | entity.entity_id | | Y |
| first_name | Text | | | | Y |
| last_name | Text | | | | |

### relationship
Types of relationships for a person

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| relationship_id | UUID | PK | | | Y |
| name | Enum | | | "Student", "Faculty", "Entrepreneur", "Donor", "Alumnus" | |

### person_relationship
Relates a person with relationship types

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| person_id | UUID | PK, FK | person.person_id | | Y |
| relationship_id | UUID | PK, FK | relationship.relationship_id | | Y |

### company
A company (startup, VC, etc.)

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| company_id | UUID | PK, FK | entity.entity_id | | Y |
| name | Text | | | | Y |
| description | Text | | | | |
| pitchbook_url | Text | | | | |
| total_raised | Num | | | | |
| last_valuation | Num | | | | |
| past_fundraising | Text | | | Past fundraising activity | |

### vc
A venture capital firm

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| vc_id | UUID | PK, FK | entity.entity_id | | Y |
| name | Text | | | | Y |
| aum | Num | | | | |

### email
An email address (each entity has one or many emails)

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| email_id | UUID | PK | | | Y |
| entity_id | UUID | FK | entity.entity_id | 1-many relationship with entity | Y |
| email_address | Text | | | | Y |
| is_primary | Bool | | | | Y |

### phone
A phone number (each person has one or many phone numbers)

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| phone_id | UUID | PK | | | Y |
| person_id | UUID | FK | person.person_id | 1-many relationship with person | |
| phone_number | Text | | | | Y |
| is_primary | Bool | | | | Y |

### user
A user of this CRM platform. Stores profile info and various metadata

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| user_id | UUID | PK, FK | person.person_id | A user is a person | Y |
| email | Text | | | | |
| avatar_url | Text | | | | |
| role | Text | | | | |
| created | Date | | | | Y |
| investments_view | Enum | | | "List" or "Kanban" | |
| alumni_view | Enum | | | "List" or "Kanban" | |
| members_view | Enum | | | "List" or "Kanban" | |
| marketing_view | Enum | | | "List" or "Kanban" | |

### industry
A specific industry segment (SaaS, healthcare, real estate, etc.)

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| industry_id | UUID | PK | | | Y |
| name | Text | | | | Y |

### company_industry
Relates an industry with a company

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| industry_id | UUID | PK, FK | industry.industry_id | | Y |
| company_id | UUID | PK, FK | company.company_id | | Y |

### person_company
Relates a person and a company, along with their title in that context

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| person_id | UUID | PK, FK | person.person_id | | Y |
| company_id | UUID | PK, FK | company.company_id | | Y |
| title | Text | | | | |

### person_vc
Relates a person and a vc, along with their title in that context

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| person_id | UUID | PK, FK | person.person_id | | Y |
| vc_id | UUID | PK, FK | vc.vc_id | | Y |
| title | Text | | | | |

### opportunity
A deal, outreach, member, or campaign

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| opportunity_id | UUID | PK | | | Y |
| name | Text | | | | |
| type | Enum | | | "Deal", "Outreach", "Member", "Campaign" | Y |
| created | Date | | | | Y |
| last_updated | Date | | | | Y |
| notes | Text | | | | |
| current_stage_id | UUID | FK | stage.stage_id | Points to stage that is currently active | Y |

### opportunity_user
Assigns fellows (users) to opportunities

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| opportunity_id | UUID | PK, FK | opportunity.opportunity_id | | Y |
| user_id | UUID | PK, FK | user.user_id | | Y |

### deal
An opportunity that represents a deal within the investment pipeline

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| deal_id | UUID | PK, FK | opportunity.opportunity_id | | Y |
| company_id | UUID | FK | company.company_id | | Y |
| drive_url | Text | | | | |
| lead_investor | UUID | FK | vc.vc_id | | |
| commitment | Num | | | | |
| round_size | Num | | | | |
| premoney_valuation | Num | | | | |
| postmoney_valuation | Num | | | | |
| closed_date | Date | | | | |

### outreach
An opportunity that represents a person within the outreach pipeline

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| outreach_id | UUID | PK, FK | opportunity.opportunity_id | | Y |
| person_id | UUID | FK | person.person_id | | Y |
| follow_up_date | Date | | | | |

### member
An opportunity that represents a member (prospective, past, current) within the membership pipeline

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| member_id | UUID | PK, FK | opportunity.opportunity_id | | Y |
| person_id | UUID | FK | person.person_id | | Y |
| role | Enum | | | "Fellow", "Advisor", "Faculty" | |
| start_date | Date | | | | |
| end_date | Date | | | | |

### campaign
An opportunity that represents a campaign within the marketing pipeline

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| campaign_id | UUID | PK, FK | opportunity.opportunity_id | | Y |
| due_date | Date | | | | |
| url | Text | | | | |

### coinvestors
VC firms that participate in a deal with the AVF

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| vc_id | UUID | PK, FK | vc.vc_id | | Y |
| deal_id | UUID | PK, FK | deal.deal_id | | Y |

### stage
The name and characteristics of a stage in a pipeline

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| stage_id | UUID | PK | | | Y |
| name | Text | | | | Y |
| type | Enum | | | "Deal", "Outreach", "Member" or "Campaign" | Y |
| order | Num | | | Determines the sequence in which stages occur | Y |

### opportunity_stage
Relates an opportunity with a stage

| Attribute | Data Type | Key Type | FK Reference | Description | Required |
|-----------|-----------|----------|--------------|-------------|----------|
| opportunity_id | UUID | PK, FK | opportunity.opportunity_id | | Y |
| stage_id | UUID | PK, FK | stage.stage_id | | Y |