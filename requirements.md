### Backend Requirement Specifications for Airbnb Platform
This document outlines the technical and functional requirements for three key backend features: User Authentication, Property Management, and Booking System. Each feature includes API endpoints, input/output specifications, validation rules, and performance criteria.
### 1. User Authentication
## Overview
The User Authentication feature enables users (Guests, Hosts, Admins) to register, log in, log out, and reset passwords securely, with role-based access control.
### Functional Requirements

## Sign Up: Allow users to create an account with name, email, password, and role.
## Login: Authenticate users with email and password.
## Logout: Invalidate the user’s session or token.
## Password Reset: Allow users to request and confirm password resets via email.
## Role-Based Access: Restrict actions based on user roles (Guest, Host, Admin).

### API Endpoints

## POST /auth/signup

Description: Register a new user.
Input:{
  "name": "string",
  "email": "string",
  "password": "string",
  "role": "string" // "guest", "host", or "admin"
}


Output:
Success (201 Created):{
  "userId": "string",
  "name": "string",
  "email": "string",
  "role": "string",
  "message": "User created successfully"
}



## Validation Rules:
Name: Required, 2–50 characters, alphanumeric with spaces.
Email: Required, valid email format (e.g., user@example.com).
Password: Required, 8–20 characters, at least one uppercase, one lowercase, one number, one special character.
Role: Required, must be “guest”, “host”, or “admin”.




## POST /auth/login

Description: Authenticate a user and issue a JWT token.
Input:{
  "email": "string",
  "password": "string"
}


Output:
Success (200 OK):{
  "userId": "string",
  "token": "string",
  "role": "string",
  "message": "Login successful"
}



## Validation Rules:
Email: Required, valid email format.
Password: Required, matches hashed password in database.




## POST /auth/password-reset

Description: Request a password reset link via email.
Input:{
  "email": "string"
}


Output:
Success (200 OK):{
  "message": "Password reset link sent"
}


## Validation Rules:
Email: Required, valid email format, exists in database.






### 2. Property Management
## Overview
The Property Management feature allows Hosts to create, edit, and delete property listings, and all users to browse and search properties based on location, price, and availability.
Functional Requirements

Create Listing: Hosts create a property with details (name, location, description, price).
Edit Listing: Hosts update existing property details.
Delete Listing: Hosts soft-delete their properties.
Browse Listings: All users view available properties.
Search & Filter: Users filter properties by location, price, and availability.

### API Endpoints

## POST /properties

Description: Create a new property listing (Host only).
Input (requires JWT in Authorization header):{
  "name": "string",
  "location": "string",
  "description": "string",
  "pricePerNight": "number",
  "availability": ["string"] // Array of dates (YYYY-MM-DD)
}


Output:
Success (201 Created):{
  "propertyId": "string",
  "name": "string",
  "location": "string",
  "description": "string",
  "pricePerNight": "number",
  "message": "Property created"
}


## Validation Rules:
Name: Required, 5–100 characters.
Location: Required, valid format (e.g., “City, Country”).
Description: Optional, max 500 characters.
PricePerNight: Required, positive number, max 10000.
Availability: Optional, valid dates in YYYY-MM-DD format.




## PUT /properties/:id

Description: Update an existing property (Host only).
Input (requires JWT):{
  "name": "string",
  "location": "string",
  "description": "string",
  "pricePerNight": "number",
  "availability": ["string"]
}


Output:
Success (200 OK):{
  "propertyId": "string",
  "name": "string",
  "location": "string",
  "description": "string",
  "pricePerNight": "number",
  "message": "Property updated"
}


## Validation Rules:
Host must own the property.


## DELETE /properties/:id

Description: Soft-delete a property (Host only).
Input: None (requires JWT).
Output:
Success (200 OK):{
  "message": "Property deleted"
}


## Validation Rules:
Property ID: Valid, exists, owned by the Host.
JWT: Valid, Host role.



## GET /properties

Description: Browse or search properties.
Input: Query parameters:?location=string&minPrice=number&maxPrice=number&startDate=string&endDate=string


Output:
Success (200 OK):{
  "properties": [
    {
      "propertyId": "string",
      "name": "string",
      "location": "string",
      "pricePerNight": "number"
    }
  ],
  "total": "number"
}





## Validation Rules:
Location: Optional, string.
MinPrice/MaxPrice: Optional, positive numbers.
StartDate/EndDate: Optional, valid YYYY-MM-DD format.






### 3. Booking System
## Overview
The Booking System allows Guests to create bookings, check availability, and view their bookings, while Hosts view bookings for their properties. Bookings are tied to availability and payments.
Functional Requirements

Create Booking: Guests select dates, verify availability, and create a booking.
Check Availability: System ensures property is available for requested dates.
View Bookings: Guests view their bookings; Hosts view bookings for their properties.
Status Management: Bookings have statuses (pending, confirmed, canceled).

## API Endpoints

## POST /bookings

Description: Create a booking (Guest only).
Input (requires JWT):{
  "propertyId": "string",
  "startDate": "string", // YYYY-MM-DD
  "endDate": "string", // YYYY-MM-DD
  "guests": "number"
}


Output:
Success (201 Created):{
  "bookingId": "string",
  "propertyId": "string",
  "startDate": "string",
  "endDate": "string",
  "guests": "number",
  "totalPrice": "number",
  "status": "pending",
  "message": "Booking created"
}




## Validation Rules:
PropertyId: Required, valid, exists in database.
StartDate/EndDate: Required, valid YYYY-MM-DD, startDate < endDate, within availability.
Guests: Required, positive integer, within property capacity.
JWT: Valid, Guest role.




## GET /bookings

Description: View bookings (Guest or Host).
Input: Query parameters:?userId=string&role=string // "guest" or "host"


Output:
Success (200 OK):{
  "bookings": [
    {
      "bookingId": "string",
      "propertyId": "string",
      "startDate": "string",
      "endDate": "string",
      "status": "string"
    }
  ],
  "total": "number"
}



### Technical Requirements

## Security:
JWT authentication for all endpoints.
Validate availability before booking creation.
HTTPS enforced.


## Error Handling:
Return 400 for invalid data, 403 for unauthorized access, 404 for missing properties.
Log errors for debugging.


