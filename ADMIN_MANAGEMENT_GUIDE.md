# Admin Account Management Guide

## Overview
Public registration has been removed. Only the Super Admin can create admin accounts for each campus through the new admin management interface.

## Features

### 1. Registration Removed
- Public `/register` route has been disabled
- Users can no longer create their own accounts
- All accounts must be created by the Super Admin

### 2. Admin Account Management (Super Admin Only)
Located at `/admins` in the application

#### Create New Admin Account
- Go to `/admins`
- Click "+ Create Admin Account"
- Fill in:
  - **Name**: Full name of the admin
  - **Email**: Unique email address
  - **Campus**: Select the admin's campus
  - **Password**: Minimum 8 characters (must confirm)
- Click "Create Admin"

#### Edit Admin Account
- Go to `/admins`
- Click "Edit" on the admin you want to modify
- Update any field
- Password is optional (leave blank to keep current)
- Click "Update Admin"

#### Delete Admin Account
- Go to `/admins`
- Click "Delete" on the admin account
- Confirm deletion

## Campus Options
Admin accounts can be assigned to:
- Urdaneta City Campus
- Lingayen Campus
- Binmaley Campus
- Bayambang Campus
- San Carlos City Campus
- Alaminos City Campus
- Asingan Campus
- Infanta Campus
- Sta. Maria Campus

## Permissions
- **Super Admin**: Can access `/admins` and manage all admin accounts
- **Admin**: Cannot access admin management (only their own staff)
- **Staff**: Cannot access admin management

## Database Changes
Super Admin account already exists and cannot be deleted through the UI.

### Super Admin Default Credentials
Look for the super_admin account in the users table.

## Routes

### Admin Management Routes (Super Admin Only)
- `GET /admins` → List all admin accounts
- `GET /admins/create` → Create admin form
- `POST /admins` → Store new admin
- `GET /admins/{user}/edit` → Edit admin form
- `PUT /admins/{user}` → Update admin
- `DELETE /admins/{user}` → Delete admin

## Security Notes
- Passwords are hashed using bcrypt
- Admin accounts are associated with a specific campus
- Campus filtering is automatically applied to all admin queries
- Super Admin account has `campus = null` (can see all data)
- Admin accounts have campus assigned and can only see their campus data
