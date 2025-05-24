# Video Analysis for Next.js Implementation Guidance

## Video URL
https://www.youtube.com/watch?v=b2y8s2oBAtU

## Prisma Schema
```prisma
generator client {
  provider = "prisma-client-js"
}

generator fabbrica {
  provider    = "prisma-fabbrica"
  output      = "../src/generated/fabbrica"
  noTranspile = true
}

datasource db {
  provider = "sqlserver"
  url      = env("DATABASE_URL")
  // shadowDatabaseUrl = env("SHADOW_DATABASE_URL")
}

model Appointment {
  id                       String           @id @default(dbgenerated("newid()"), map: "PK_Appointment_ID") @db.UniqueIdentifier
  type                     String           @db.VarChar(50)
  title                    String?          @db.VarChar(255)
  is_all_day               Boolean          @default(false, map: "DF_Appointment_IsAllDay")
  start_date               DateTime
  end_date                 DateTime
  location_id              String?          @db.UniqueIdentifier
  created_by               String           @db.UniqueIdentifier
  status                   String           @db.VarChar(100)
  client_group_id          String?          @db.UniqueIdentifier
  clinician_id             String?          @db.UniqueIdentifier
  appointment_fee          Decimal?
  write_off                Decimal?
  adjustable_amount        Decimal?
  service_id               String?          @db.UniqueIdentifier
  is_recurring             Boolean          @default(false, map: "DF_Appointment_IsRecurring")
  recurring_rule           String?          @db.Text
  cancel_appointments      Boolean?
  notify_cancellation      Boolean?
  recurring_appointment_id String?          @db.UniqueIdentifier
  ClientGroup              ClientGroup?     @relation(fields: [client_group_id], references: [id], onDelete: NoAction, onUpdate: NoAction, map: "FK_Appointment_ClientGroup")
  Clinician                Clinician?       @relation(fields: [clinician_id], references: [id], onDelete: NoAction, onUpdate: NoAction, map: "FK_Appointment_Clinician")
  User                     User?             @relation(fields: [created_by], references: [id], map: "FK_Appointment_User")
  Location                 Location?        @relation(fields: [location_id], references: [id], map: "FK_Appointment_Location")
  Appointment              Appointment?     @relation("AppointmentToAppointment", fields: [recurring_appointment_id], references: [id], onDelete: NoAction, onUpdate: NoAction, map: "FK_Appointment_RecurringAppointment")
  other_Appointment        Appointment[]    @relation("AppointmentToAppointment")
  PracticeService          PracticeService? @relation(fields: [service_id], references: [id], map: "FK_Appointment_PracticeService")
  AppointmentTag           AppointmentTag[]
  Invoice                  Invoice[]
  SurveyAnswers            SurveyAnswers[]
}

model AppointmentTag {
  id             String      @id(map: "PK_AppointmentTag_ID") @default(dbgenerated("newid()"), map: "DF_AppointmentTag_ID") @db.UniqueIdentifier
  appointment_id String      @db.UniqueIdentifier
  tag_id         String      @db.UniqueIdentifier
  Appointment    Appointment @relation(fields: [appointment_id], references: [id], onUpdate: NoAction, map: "FK_AppointmentTag_Appointment")
  Tag            Tag         @relation(fields: [tag_id], references: [id], onUpdate: NoAction, map: "FK_AppointmentTag_Tag")

  @@unique([appointment_id, tag_id], map: "UQ_AppointmentTag_Appointment_Tag")
  @@index([appointment_id], map: "IX_AppointmentTag_appointment_id")
  @@index([tag_id], map: "IX_AppointmentTag_tag_id")
}

model Audit {
  Id         String   @id(map: "PK_Audit_ID") @default(dbgenerated("newid()"), map: "DF_Audit_ID") @db.UniqueIdentifier
  client_id  String?  @db.UniqueIdentifier
  user_id    String?  @db.UniqueIdentifier
  datetime   DateTime @default(now(), map: "DF_Audit_Datetime") @db.DateTime
  event_type String?  @db.NChar(10)
  event_text String   @db.NVarChar(255)
  is_hipaa   Boolean  @default(false, map: "DF_Audit_IsHipaa")
  Client     Client?  @relation(fields: [client_id], references: [id], onDelete: NoAction, onUpdate: NoAction, map: "FK_Audit_Client")
  User       User?    @relation(fields: [user_id], references: [id], onDelete: NoAction, onUpdate: NoAction, map: "FK_Audit_User")
}

model Availability {
  id                    String    @id @default(dbgenerated("newid()"), map: "PK_Availability_ID") @db.UniqueIdentifier
  title                 String    @db.Text
  clinician_id          String    @db.UniqueIdentifier
  start_date            DateTime  @default(now(), map: "DF_Availability_StartDate")
  end_date              DateTime  @default(now(), map: "DF_Availability_EndDate")
  start_time            DateTime  @default(now(), map: "DF_Availability_StartTime")
  end_time              DateTime  @default(now(), map: "DF_Availability_EndTime") // Default to now(), but handle this in your application logic for the correct end time
  created_at            DateTime  @default(now(), map: "DF_Availability_CreatedAt")
  updated_at            DateTime  @updatedAt
  is_recurring          Boolean   @default(false, map: "DF_Availability_IsRecurring")
  recurring_rule        String?   @db.Text
  allow_online_requests Boolean   @default(false, map: "DF_Availability_AllowOnlineRequests")
  location              String?   @db.VarChar(255)
  Clinician             Clinician @relation(fields: [clinician_id], references: [id], onDelete: NoAction, onUpdate: NoAction, map: "FK_Availability_Clinician")

  @@index([clinician_id], map: "IX_Availability_clinician_id")
  @@index([start_time, end_time], map: "IX_Availability_time_range")
}

model Client {
  id                       String                     @id @default(dbgenerated("newid()"), map: "PK_Client_ID") @db.UniqueIdentifier
  legal_first_name         String                     @db.VarChar(100)
  legal_last_name          String                     @db.VarChar(100)
  is_waitlist              Boolean                    @default(false, map: "DF_Client_IsWaitlist")
  primary_clinician_id     String?                    @db.UniqueIdentifier
  primary_location_id      String?                    @db.UniqueIdentifier
  created_at               DateTime                   @default(now(), map: "DF_Client_CreatedAt")
  is_active                Boolean                    @default(true, map: "DF_Client_IsActive")
  preferred_name           String?                    @db.VarChar(100)
  date_of_birth            DateTime?                  @db.Date
  referred_by              String?                    @db.VarChar(200)
  Audit                    Audit[]
  Clinician                Clinician?                 @relation(fields: [primary_clinician_id], references: [id], onUpdate: NoAction, map: "FK_Client_Clinician")
  Location                 Location?                  @relation(fields: [primary_location_id], references: [id], onUpdate: NoAction, map: "FK_Client_Location")
  ClientContact            ClientContact[]
  ClientGroupMembership    ClientGroupMembership[]
  ClientReminderPreference ClientReminderPreference[]
  ClinicianClient          ClinicianClient[]
  CreditCard               CreditCard[]
  SurveyAnswers            SurveyAnswers[]
}

model ClientContact {
  id           String  @id @default(dbgenerated("newid()"), map: "PK_ClientContact_ID") @db.UniqueIdentifier
  client_id    String  @db.UniqueIdentifier
  is_primary   Boolean @default(false, map: "DF_ClientContact_IsPrimary")
  permission   String  @db.VarChar(50)
  contact_type String  @db.VarChar(50)
  type         String  @db.VarChar(50)
  value        String  @db.VarChar(255)
  Client       Client  @relation(fields: [client_id], references: [id], onUpdate: NoAction, map: "FK_ClientContact_Client")
}

model ClientGroup {
  id                    String                  @id(map: "PK_ClientGroup_ID") @db.UniqueIdentifier
  type                  String                  @db.VarChar(150)
  name                  String                  @db.VarChar(250)
  available_credit      Decimal                @default(0)
  clinician_id          String?                 @db.UniqueIdentifier
  is_active             Boolean                 @default(true, map: "DF_ClientGroup_IsActive")
  Appointment           Appointment[]
  ClientGroupMembership ClientGroupMembership[]
  Invoice               Invoice[]
  Clinician             Clinician?    @relation(fields: [clinician_id], references: [id], onUpdate: NoAction, map: "FK_ClientGroup_Clinician")
}

model ClientGroupMembership {
  client_group_id            String      @default(dbgenerated("newid()"), map: "DF_ClientGroupMembership_ID") @db.UniqueIdentifier
  client_id                  String      @db.UniqueIdentifier
  role                       String?     @db.VarChar(50)
  created_at                 DateTime    @default(now(), map: "DF_ClientGroupMembership_CreatedAt")
  is_contact_only            Boolean     @default(false, map: "DF_ClientGroupMembership_IsContactOnly")
  is_responsible_for_billing Boolean?
  Client                     Client      @relation(fields: [client_id], references: [id], onUpdate: NoAction, map: "FK_ClientGroupMembership_Client")
  ClientGroup                ClientGroup @relation(fields: [client_group_id], references: [id], onUpdate: NoAction, map: "FK_ClientGroupMembership_ClientGroup")

  @@id([client_group_id, client_id], map: "PK_ClientGroupMembership_ID")
  @@index([client_id], map: "IX_ClientGroupMembership_client_id")
}

model ClientReminderPreference {
  id            String  @id @default(dbgenerated("newid()"), map: "PK_ClientReminderPreference_ID") @db.UniqueIdentifier
  client_id     String  @db.UniqueIdentifier
  reminder_type String  @db.VarChar(100)
  is_enabled    Boolean @default(true, map: "DF_ClientReminderPreference_IsEnabled")
  Client        Client  @relation(fields: [client_id], references: [id], onDelete: Cascade, map: "FK_ClientReminderPreference_Client")

  @@unique([client_id, reminder_type])
}

model BillingAddress {
  id           String     @id @default(dbgenerated("newid()"), map: "PK_BillingAddress_ID") @db.UniqueIdentifier
  street       String     @db.VarChar(255)
  city         String     @db.VarChar(100)
  state        String     @db.VarChar(50)
  zip          String     @db.VarChar(20)
  type         String     @db.VarChar(50) // 'business' or 'client'
  clinician_id String     @db.UniqueIdentifier
  Clinician    Clinician  @relation("ManyBillingAddresses", fields: [clinician_id], references: [id], onDelete: Cascade, map: "FK_BillingAddress_Clinician")

  @@index([clinician_id], map: "IX_BillingAddress_clinician_id")
  @@unique([clinician_id, type], map: "UQ_BillingAddress_Clinician_Type")
}

model Clinician {
  id                String              @id @default(dbgenerated("newid()"), map: "PK_Clinician_ID") @db.UniqueIdentifier
  user_id           String              @unique @db.UniqueIdentifier
  address           String              @db.Text
  percentage_split  Float
  is_active         Boolean             @default(true, map: "DF_Clinician_IsActive")
  first_name        String              @db.VarChar(100)
  last_name         String              @db.VarChar(100)
  Appointment       Appointment[]
  Client            Client[]
  User              User                @relation(fields: [user_id], references: [id], map: "FK_Clinician_User")
  appointmentLimits AppointmentLimit[]
  ClinicianClient   ClinicianClient[]
  ClinicianLocation ClinicianLocation[]
  ClinicianServices ClinicianServices[]
  Invoice           Invoice[]
  ClientGroup       ClientGroup[]
  Availability      Availability[]
  billingAddresses  BillingAddress[]    @relation("ManyBillingAddresses")
}

model ClinicianClient {
  client_id     String    @db.UniqueIdentifier
  clinician_id  String    @db.UniqueIdentifier
  is_primary    Boolean   @default(false, map: "DF_ClinicianClient_IsPrimary")
  assigned_date DateTime  @default(now(), map: "DF_ClinicianClient_AssignedDate")
  Client        Client    @relation(fields: [client_id], references: [id], onUpdate: NoAction, map: "FK_ClinicianClient_Client")
  Clinician     Clinician @relation(fields: [clinician_id], references: [id], onUpdate: NoAction, map: "FK_ClinicianClient_Clinician")

  @@id([client_id, clinician_id], map: "PK_ClinicianClient_ID")
}

model ClinicianLocation {
  clinician_id String    @db.UniqueIdentifier
  location_id  String    @db.UniqueIdentifier
  is_primary   Boolean   @default(false, map: "DF_ClinicianLocation_IsPrimary")
  Clinician    Clinician @relation(fields: [clinician_id], references: [id], map: "FK_ClinicianLocation_Clinician")
  Location     Location  @relation(fields: [location_id], references: [id], map: "FK_ClinicianLocation_Location")

  @@id([clinician_id, location_id], map: "PK_ClinicianLocation_ID")
}

model ClinicianServices {
  clinician_id    String          @db.UniqueIdentifier
  service_id      String          @db.UniqueIdentifier
  custom_rate     Decimal?
  is_active       Boolean         @default(true, map: "DF_ClinicianServices_IsActive")
  Clinician       Clinician       @relation(fields: [clinician_id], references: [id], map: "FK_ClinicianServices_Clinician")
  PracticeService PracticeService @relation(fields: [service_id], references: [id], map: "FK_ClinicianServices_PracticeService")

  @@id([clinician_id, service_id], map: "PK_ClinicianServices_ID")
}

model CreditCard {
  id              String    @id @default(dbgenerated("newid()"), map: "PK_CreditCard_ID") @db.UniqueIdentifier
  client_id       String    @db.UniqueIdentifier
  card_type       String    @db.VarChar(50)
  last_four       String    @db.VarChar(4)
  expiry_month    Int
  expiry_year     Int
  cardholder_name String    @db.VarChar(100)
  is_default      Boolean   @default(false, map: "DF_CreditCard_IsDefault")
  billing_address String?   @db.Text
  token           String?   @db.VarChar(255)
  created_at      DateTime  @default(now(), map: "DF_CreditCard_CreatedAt")
  Client          Client    @relation(fields: [client_id], references: [id], onUpdate: NoAction, map: "FK_CreditCard_Client")
  Payment         Payment[]
}

model Invoice {
  id                    String       @id @default(dbgenerated("newid()"), map: "PK_Invoice_ID") @db.UniqueIdentifier
  invoice_number        String       @db.VarChar(50)
  client_group_id       String?      @db.UniqueIdentifier
  appointment_id        String?      @db.UniqueIdentifier
  clinician_id          String?      @db.UniqueIdentifier
  issued_date           DateTime     @default(now(), map: "DF_Invoice_IssuedDate")
  due_date              DateTime
  amount                Decimal      @db.Decimal(10, 2)
  status                String       @db.VarChar(50)
  type                  String       @db.VarChar(50)
  client_info           String?      @db.Text
  provider_info         String?      @db.Text
  service_description   String?      @db.Text
  notes                 String?      @db.Text
  Appointment           Appointment? @relation(fields: [appointment_id], references: [id], onDelete: NoAction, onUpdate: NoAction, map: "FK_Invoice_Appointment")
  ClientGroup           ClientGroup? @relation(fields: [client_group_id], references: [id], onDelete: NoAction, onUpdate: NoAction, map: "FK_Invoice_ClientGroup")
  Clinician             Clinician?   @relation(fields: [clinician_id], references: [id], onUpdate: NoAction, map: "FK_Invoice_Clinician")
  Payment               Payment[]
}

model Location {
  id                String              @id @default(dbgenerated("newid()"), map: "PK_Location_ID") @db.UniqueIdentifier
  name              String              @db.VarChar(255)
  address           String              @db.Text
  street            String?             @db.VarChar(255)
  city              String?             @db.VarChar(100)
  state              String?             @db.VarChar(100)
  zip               String?             @db.VarChar(20)
  color             String?             @db.VarChar(50)
  is_active         Boolean             @default(true, map: "DF_Location_IsActive")
  Appointment       Appointment[]
  Client            Client[]
  ClinicianLocation ClinicianLocation[]
}

model Payment {
  id             String      @id @default(dbgenerated("newid()"), map: "PK_Payment_ID") @db.UniqueIdentifier
  invoice_id     String      @db.UniqueIdentifier
  payment_date   DateTime    @default(now(), map: "DF_Payment_PaymentDate")
  amount         Decimal     @db.Decimal(10, 2)
  credit_applied Decimal?    @db.Decimal(10, 2)
  credit_card_id String?     @db.UniqueIdentifier
  transaction_id String?     @db.VarChar(100)
  status         String      @db.VarChar(50)
  response       String?     @db.Text
  CreditCard     CreditCard? @relation(fields: [credit_card_id], references: [id], onDelete: NoAction, onUpdate: NoAction, map: "FK_Payment_CreditCard")
  Invoice        Invoice     @relation(fields: [invoice_id], references: [id], onUpdate: NoAction, map: "FK_Payment_Invoice")
}

model PracticeService {
  id                String              @id @default(dbgenerated("newid()"), map: "PK_PracticeService_ID") @db.UniqueIdentifier
  code              String              @db.VarChar(50) // "Service" in UI
  description       String?             @db.Text
  rate              Decimal             @db.Decimal(10, 2)
  duration          Int                 // Default Duration (minutes)
  color             String?             @db.VarChar(7)
  is_default        Boolean             @default(false)
  bill_in_units     Boolean             @default(false)
  available_online  Boolean             @default(false)
  allow_new_clients Boolean             @default(false)
  require_call      Boolean             @default(false)
  block_before      Int                 @default(0) // Minutes before appointment
  block_after       Int                 @default(0) // Minutes after appointment
  type              String              @db.VarChar(255)
  Appointment       Appointment[]
  ClinicianServices ClinicianServices[]
}

model Role {
  id       String     @id @default(dbgenerated("newid()"), map: "PK_Role_ID") @db.UniqueIdentifier
  name     String     @unique @db.VarChar(255)
  UserRole UserRole[]
}

model SurveyAnswers {
  id             String         @id(map: "PK_SurveyAnswers_ID") @default(dbgenerated("newid()"), map: "DF_SurveyAnswers_ID") @db.UniqueIdentifier
  template_id    String         @db.UniqueIdentifier
  client_id      String         @db.UniqueIdentifier
  content        String?        @db.Text
  frequency      String?        @db.NChar(10)
  completed_at   DateTime?
  assigned_at    DateTime       @default(now(), map: "DF_SurveyAnswers_AssignedAt")
  expiry_date    DateTime?
  status         String         @db.VarChar(100)
  appointment_id String?        @db.UniqueIdentifier
  is_signed      Boolean?
  is_locked      Boolean?
  Client         Client         @relation(fields: [client_id], references: [id], onUpdate: NoAction, map: "FK_SurveyAnswers_Client")
  SurveyTemplate SurveyTemplate @relation(fields: [template_id], references: [id], onUpdate: NoAction, map: "FK_SurveyAnswers_SurveyTemplate")
  Appointment    Appointment?   @relation(fields: [appointment_id], references: [id], onDelete: NoAction, onUpdate: NoAction, map: "FK_SurveyAnswers_Appointment")
}

model SurveyTemplate {
  id                 String          @id(map: "PK_SurveyTemplate_ID") @default(dbgenerated("newid()"), map: "DF_SurveyTemplate_ID") @db.UniqueIdentifier
  name               String          @db.VarChar(255)
  content            String          @db.Text
  frequency_options  String?         @db.NChar(10)
  is_active          Boolean         @default(true, map: "DF_SurveyTemplate_IsActive")
  created_at         DateTime        @default(now(), map: "DF_SurveyTemplate_CreatedAt")
  description        String?         @db.Text
  updated_at         DateTime
  type               String          @db.VarChar(100)
  requires_signature Boolean         @default(false, map: "DF_SurveyTemplate_RequiresSignature")
  SurveyAnswers      SurveyAnswers[]
}

model sysdiagrams {
  name         String @db.NVarChar(128)
  principal_id Int
  diagram_id   Int    @id(map: "PK_sysdiagrams_ID") @default(autoincrement())
  version      Int?
  definition   Bytes?

  @@unique([principal_id, name], map: "UK_principal_name")
}

model Tag {
  id             String           @id(map: "PK_Tag_ID") @default(dbgenerated("newid()"), map: "DF_Tag_ID") @db.UniqueIdentifier
  name           String           @db.NVarChar(100)
  color          String?          @db.NVarChar(50)
  AppointmentTag AppointmentTag[]
}

model User {
  id            String         @id @default(dbgenerated("newid()")) @db.UniqueIdentifier
  email         String         @unique @db.VarChar(255)
  password_hash String         @db.VarChar(255)
  last_login    DateTime?
  Appointment   Appointment[]
  Audit         Audit[]
  Clinician     Clinician?
  UserRole      UserRole[]
 date_of_birth  DateTime? @db.Date
  phone         String?   @db.VarChar(20)
  profile_photo String?   @db.VarChar(500)
  clinicalInfos ClinicalInfo[]
  practiceInformation PracticeInformation[]
}

model UserRole {
  user_id String @db.UniqueIdentifier
  role_id String @db.UniqueIdentifier
  Role    Role   @relation(fields: [role_id], references: [id], map: "FK_UserRole_Role")
  User    User   @relation(fields: [user_id], references: [id], map: "FK_UserRole_User")

  @@id([user_id, role_id], map: "PK_UserRole_ID")
}

model ClinicalInfo {
  id           Int    @id @default(autoincrement())
  speciality   String
  taxonomy_code String
  NPI_number    Float
  user_id       String @db.UniqueIdentifier
  licenses      License[]
  User User @relation(fields: [user_id], references: [id], map: "FK_clinicalInfo_User")
}

model License {
  id              Int      @id @default(autoincrement())
  clinical_info_id Int
  license_type    String   // License type
  license_number  String   // License number
  expiration_date DateTime     // Use Date type for expiration date
  state           String   // State
  clinicalInfo    ClinicalInfo @relation(fields: [clinical_info_id], references: [id]) // Establishing the relation
}

model PracticeInformation {
  id            String   @id @default(dbgenerated("newid()")) @db.UniqueIdentifier
  practice_name String
  practice_email String
  time_zone     String
  practice_logo String
  phone_numbers String
  tele_health   Boolean
  user_id       String @db.UniqueIdentifier
  User User @relation(fields: [user_id], references: [id], map: "FK_practiceInformation_User")
}

model AppointmentLimit {
  id           String   @id @default(dbgenerated("newid()")) @db.UniqueIdentifier
  date         DateTime @db.Date
  max_limit    Int      @default(10)
  clinician_id String   @db.UniqueIdentifier

  // Relationships
  Clinician Clinician @relation(fields: [clinician_id], references: [id], onDelete: Cascade, onUpdate: NoAction)

  @@unique([date, clinician_id], name: "UQ_AppointmentLimit_Date_Clinician")
  @@index([clinician_id], name: "IX_AppointmentLimit_clinician_id")
  @@index([date], name: "IX_AppointmentLimit_date")
}

model Product {
  id    String  @id @default(dbgenerated("newid()")) @db.UniqueIdentifier
  name  String  @db.VarChar(255)
  price Decimal @db.Decimal(10, 2)
}

```

## Task Overview
Analyze the video accessible via the provided URL where a user  shows the functionality of the web application for reference. Generate structured JSON with implementation guidance for developers.

## Technical Stack Context
- Framework: Next.js with App Router
- API: REST endpoints
- Database: Prisma ORM (schema provided above)
- UI: Shadcn components
- TypeScript: Used throughout the application

## JSON Structure
For each unique Figma screen, create a JSON object with the following properties and combine all objects into a single array:

```json
[
  {
    "wireframeId": "path/to/screen",
    "wireframeTitle": "Short title for screen",
    "functionality": "Description of functionality",
    "implementation": {
      "route": {
        "path": "Suggested Next.js route path",
        "layout": "Whether a layout file is needed",
        "components": ["Suggested main components"]
      },
      "api": {
        "endpoints": [
          {
            "path": "/api/path",
            "method": "GET|POST|PUT|DELETE",
            "purpose": "What this endpoint does",
            "requestBody": "Description of expected request data",
            "responseData": "Description of returned data"
          }
        ]
      },
      "data": {
        "tables": ["Prisma model names needed"],
        "queries": ["High-level description of required database operations"]
      },
      "ui": {
        "suggestedComponents": ["Relevant Shadcn components"],
        "interactions": ["Key user interactions"]
      }
    }
  }
]
```

## Field Guidelines

### wireframeId
- Should uniquely identify the screen
- Format as a URL-compatible path
- If screen corresponds to a Simple Practice webpage, derive path from that URL
- For popups, add suffix like "-popup-[purpose]"
- For workflow steps, add suffix like "-step-[number]"

### wireframeTitle
- Provide a concise title describing the functionality
- Think of it as a page title in HTML
- Maximum 5-7 words

### functionality
- Brief description of what the screen does
- Extract from verbal explanation in the video or from the Simple Practice reference
- Focus on primary user actions and purpose

### route
- Provide suggested App Router path based on functionality and wireframeId
- Indicate if special layouts or route groups are needed
- List main components needed (page.tsx, layout.tsx, loading.tsx, etc.)

### api
- Suggest REST endpoints needed to support the functionality
- For each endpoint, specify HTTP method, purpose, and data structure
- Keep descriptions high-level but specific enough to guide implementation

### data
- Reference Prisma models that will be needed
- Suggest high-level queries (create, read, update, delete operations)
- Note any potential performance considerations

### ui
- Recommend appropriate Shadcn components
- List key user interactions that need to be implemented

## Important Notes
- Only generate JSON objects for screens shown in Figma
- When the user shows Simple Practice, use this as reference for understanding functionality
- If a field cannot be determined with high confidence, provide a descriptive placeholder
- Track screens to ensure the same wireframeId is used for repeated screens
- Focus on high-level guidance, not detailed implementation
- Provide enough detail for developers to understand what needs to be built
- Consider standard Next.js patterns and conventions

## Example Output

```json
[
  {
    "wireframeId": "calendar/overview",
    "wireframeTitle": "Calendar View",
    "functionality": "Displays monthly calendar with appointment slots and allows booking new appointments",
    "implementation": {
      "route": {
        "path": "/calendar",
        "layout": "Uses a dashboard layout with navigation sidebar",
        "components": ["page.tsx", "CalendarView.tsx", "AppointmentSlot.tsx"]
      },
      "api": {
        "endpoints": [
          {
            "path": "/api/appointments",
            "method": "GET",
            "purpose": "Fetch all appointments for a date range",
            "requestBody": "Query parameters for date range and filters",
            "responseData": "Array of appointment objects with time slots and patient info"
          },
          {
            "path": "/api/appointments",
            "method": "POST",
            "purpose": "Create a new appointment",
            "requestBody": "Appointment details including patient, date, time, and type",
            "responseData": "Created appointment object with ID"
          }
        ]
      },
      "data": {
        "tables": ["Appointment", "User", "Patient"],
        "queries": [
          "Find all appointments within date range with relations",
          "Create new appointment with patient reference"
        ]
      },
      "ui": {
        "suggestedComponents": ["Calendar", "Button", "Dialog", "Form", "Select"],
        "interactions": [
          "Click day to view appointments",
          "Click time slot to book appointment",
          "Drag and drop to reschedule"
        ]
      }
    }
  },
  {
    "wireframeId": "settings/notifications-popup",
    "wireframeTitle": "Notification Settings",
    "functionality": "Configure which calendar events trigger notifications and delivery methods",
    "implementation": {
      "route": {
        "path": "/settings/notifications",
        "layout": "Uses settings layout with tabs",
        "components": ["page.tsx", "NotificationForm.tsx"]
      },
      "api": {
        "endpoints": [
          {
            "path": "/api/users/notifications/preferences",
            "method": "GET",
            "purpose": "Fetch user notification preferences",
            "requestBody": "None",
            "responseData": "Object with notification settings for different event types"
          },
          {
            "path": "/api/users/notifications/preferences",
            "method": "PUT",
            "purpose": "Update notification preferences",
            "requestBody": "Updated notification settings object",
            "responseData": "Updated preferences object"
          }
        ]
      },
      "data": {
        "tables": ["User", "NotificationPreference"],
        "queries": [
          "Get notification preferences for current user",
          "Update notification preferences"
        ]
      },
      "ui": {
        "suggestedComponents": ["Form", "Switch", "Checkbox", "Card", "Button"],
        "interactions": [
          "Toggle notification types on/off",
          "Select delivery methods",
          "Save preferences"
        ]
      }
    }
  }
]
```
