# Requirement Specifications for Backend Features

1. **User Authentication**

   ### Feature 1.1: User Registration with Email

   - **Description**: Allows a new user to create an account using their email and a password. The system will hash the password before storing it. Roles are assigned as "guest" by default.
   - **Endpoints**:
     - `POST /api/register`
   - **Request Body**:
     ```json
     {
       "first_name": "John",
       "last_name": "Doe",
       "email": "john.doe@email.com",
       "password": "SecurePassword123$",
       "phone_number": "+1234567890",
       "role": "guest"
     }
     ```
   - **Response**:
     - `201 Created` on success with user details (excluding password).
     - **Response Body**:
       ```json
       {
         "user_id": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
         "first_name": "John",
         "last_name": "Doe",
         "email": "john.doe@email.com",
         "role": "guest",
         "phone_number": "+1234567890",
         "created_at": "2024-10-01T12:00:00Z"
       }
       ```
     - `400 Bad Request` if email already exists or validation fails.
     - `409 Conflict` if email is already registered.
   - **Validation**:
     - Email must be in a valid format.
     - Password must be at least 8 characters, include uppercase, lowercase, number, and special character.
     - Phone number must be in a valid format.
   - **Security**:
     - Passwords must be hashed using bcrypt before storage.
     - Implement rate limiting to prevent brute-force attacks.
   - **Performance**:
     - The registration endpoint should respond within 400ms under normal load conditions.

   ### Feature 1.2: User Login & JWT Token Generation

   - **Description**: Allows an existing user to log in using their email and password. Returns a JWT token for authenticated requests.
   - **Endpoints**:
     - `POST /api/login`
   - **Request Body**:
     ```json
     {
       "email": "john.doe@email.com",
       "password": "SecurePassword123$"
     }
     ```
   - **Response**:
     - `200 OK` on success with JWT token.
     - **Response Body**:
       ```json
       {
         "first_name": "John",
         "last_name": "Doe",
         "email": "john.doe@email.com",
         "role": "guest",
         "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
         "refresh_token": "dGhpc0lzQXJlZnJlc2hUb2tlbg=="
       }
       ```
     - `401 Unauthorized` if credentials are invalid.
     - `400 Bad Request` if validation fails.
   - **Validation**:
     - Email must be in a valid format.
     - Password must not be empty.
   - **Security**:
     - Use HTTPS for all requests.
     - JWT tokens should expire after 1 hour.
     - Implement refresh tokens for session management.
   - **Performance**:
     - The login endpoint should respond within 300ms under normal load conditions.

2. Property Management

   ### Feature 2.1: Create Property Listing

   - **Description**: Allows an authenticated host to create a new property listing with details such as title, description, price, and location.
   - **Endpoints**:
     - `POST /api/properties`
   - **Input Specification**:
     - **Headers**:
       - `Authorization: Bearer <JWT_TOKEN>`
     - **Content-Type**: `application/json`
     - **Request Body**:
       ```json
       {
         "title": "Cozy Apartment in Downtown",
         "description": "A comfortable and modern apartment located in the heart of the city.",
         "price_per_night": 120.0,
         "address": {
           "street": "123 Main St",
           "city": "Metropolis",
           "state": "NY",
           "zip_code": "10001",
           "country": "USA"
         },
         "amenities": ["Wi-Fi", "Air Conditioning", "Kitchen"],
         "availability": {
           "start_date": "2024-11-01",
           "end_date": "2024-12-31"
         },
         "host_id": "host-uuid-1234"
       }
       ```
   - **Response**:
     - `201 Created` on success with property details.
     - **Response Body**:
       ```json
       {
         "property_id": "p1b2c3d4-e5f6-7890-1234-567890abcdef",
         "title": "Cozy Apartment in Downtown",
         "description": "A comfortable and modern apartment located in the heart of the city.",
         "price_per_night": 120.0,
         "address": {
           "street": "123 Main St",
           "city": "Metropolis",
           "state": "NY",
           "zip_code": "10001",
           "country": "USA"
         },
         "amenities": ["Wi-Fi", "Air Conditioning", "Kitchen"],
         "availability": {
           "start_date": "2024-11-01",
           "end_date": "2024-12-31"
         },
         "host_id": "host-uuid-1234",
         "created_at": "2024-10-01T12:00:00Z"
       }
       ```
     - `400 Bad Request` if validation fails.
     - `401 Unauthorized` if the user is not authenticated or not a host.
   - **Validation**:
     - Title must be between 10 and 100 characters.
     - Description must be between 20 and 100, required and non-empty strings.
     - Price must be a positive number.
     - Address fields must be non-empty strings.
     - Availability dates must be valid and the end date must be after the start date.
   - **Security**:
     - Ensure only authenticated hosts can create listings.
     - Validate JWT token on each request.
   - **Performance**:
     - The property creation endpoint should respond within 500ms under normal load conditions.

   ### Feature 2.2: Search and Filter Property Listings

   - **Description**: Allows users to search for property listings based on location, price range, and amenities.
   - **Endpoints**:
     - `GET /api/properties`
   - **Input Specification**:
     - **Query Parameters**:
       - `location` (string, optional): City or zip code to search properties.
       - `min_price` (number, optional): Minimum price per night.
       - `max_price` (number, optional): Maximum price per night.
       - `amenities` (string, optional): Comma-separated list of required amenities (e.g., "Wi-Fi,Pool").
       - `sort_by` (string, optional): "price_asc", "price_desc", "rating_desc".
       - `page` (number, optional): Page number for pagination (default is 1).
       - `limit` (number, optional): Number of listings per page (default is 10, max is 50).
   - **Response**:
     - `200 OK` on success with a list of property listings.
     - **Response Body**:
       ```json
       {
         "pagination": {
           "current_page": 1,
           "total_pages": 5,
           "total_items": 45,
           "items_per_page": 10
         },
         "properties": [
           {
             "property_id": "p1b2c3d4-e5f6-7890-1234-567890abcdef",
             "title": "Cozy Apartment in Downtown",
             "description": "A comfortable and modern apartment located in the heart of the city.",
             "price_per_night": 120.0,
             "address": {
               "street": "123 Main St",
               "city": "Metropolis",
               "state": "NY",
               "zip_code": "10001",
               "country": "USA"
             },
             "amenities": ["Wi-Fi", "Air Conditioning", "Kitchen"],
             "average_rating": 4.5,
             "host_id": "host-uuid-1234"
           }
           // More properties...
         ]
       }
       ```
       - `400 Bad Request` if validation fails.
       - `401 Unauthorized` if the user is not authenticated.
   - **Validation**:
     - Location must be a non-empty string if provided.
     - Price values must be positive numbers if provided.
     - Amenities must be a comma-separated list of valid strings if provided.
     - Sort_by must be one of the allowed values if provided.
     - Page and limit must be positive integers.
   - **Security**:
     - Ensure only authenticated users can access the search endpoint.
     - Validate JWT token on each request.
   - **Performance**:
     - The search endpoint should respond within 600ms under normal load conditions. Database queries should be optimized for performance.

   ### Feature 2.3: Update Property Listing

   - **Description**: Allows an authenticated host to update details of their existing property listing.
   - **Endpoints**:
     - `PUT /api/properties/{property_id}`
   - **Input Specification**:
     - **Headers**:
       - `Authorization: Bearer <JWT_TOKEN>`
     - **Content-Type**: `application/json`
     - **Path Parameters**:
       - `property_id` (string, required): The unique identifier of the property to be updated.
     - **Request Body**:
       ```json
       {
         "title": "Updated Cozy Apartment in Downtown",
         "description": "An updated description of the comfortable and modern apartment.",
         "price_per_night": 130.0,
         "address": {
           "street": "123 Main St",
           "city": "Metropolis",
           "state": "NY",
           "zip_code": "10001",
           "country": "USA"
         },
         "amenities": ["Wi-Fi", "Air Conditioning", "Kitchen", "Pool"],
         "availability": {
           "start_date": "2024-11-01",
           "end_date": "2024-12-31"
         }
       }
       ```
   - **Response**:
     - `200 OK` on success with updated property details.
     - **Response Body**:
       ```json
       {
         "property_id": "p1b2c3d4-e5f6-7890-1234-567890abcdef",
         "title": "Updated Cozy Apartment in Downtown",
         "description": "An updated description of the comfortable and modern apartment.",
         "price_per_night": 130.0,
         "address": {
           "street": "123 Main St",
           "city": "Metropolis",
           "state": "NY",
           "zip_code": "10001",
           "country": "USA"
         },
         "amenities": ["Wi-Fi", "Air Conditioning", "Kitchen", "Pool"],
         "availability": {
           "start_date": "2024-11-01",
           "end_date": "2024-12-31"
         },
         "host_id": "host-uuid-1234",
         "updated_at": "2024-10-02T12:00:00Z"
       }
       ```
     - `400 Bad Request` if validation fails.
     - `401 Unauthorized` if the user is not authenticated or not the owner of the property.
     - `404 Not Found` if the property does not exist.
   - **Validation**:
     - Title must be between 10 and 100 characters if provided.
     - Description must be between 20 and 1000 characters if provided.
     - Price must be a positive number if provided.
     - Address fields must be non-empty strings if provided.
     - Availability dates must be valid and the end date must be after the start date if provided.
   - **Security**:
     - Ensure only authenticated hosts can update their listings.
     - Validate JWT token on each request.
     - Verify that the authenticated user is the owner of the property.
   - **Performance**:
     - The property update endpoint should respond within 500ms under normal load conditions. Database updates should be optimized for performance.

3. Booking Management

   ### Feature 3.1: Create Booking

   - **Description**: Allows an authenticated guest to create a booking for a property listing.
   - **Endpoints**:
     - `POST /api/bookings`
   - **Input Specification**:
     - **Headers**:
       - `Authorization: Bearer <JWT_TOKEN>`
     - **Content-Type**: `application/json`
     - **Request Body**:
       ```json
       {
         "property_id": "p1b2c3d4-e5f6-7890-1234-567890abcdef",
         "guest_id": "guest-uuid-5678",
         "check_in": "2024-11-10",
         "check_out": "2024-11-15",
         "total_price": 600.0,
         "payment_method": "credit_card"
       }
       ```
   - **Response**:
     - `201 Created` on success with booking details.
     - **Response Body**:
       ```json
       {
         "booking_id": "b1c2d3e4-f5g6-7890-1234-567890abcdef",
         "property_id": "p1b2c3d4-e5f6-7890-1234-567890abcdef",
         "guest_id": "guest-uuid-5678",
         "check_in": "2024-11-10",
         "check_out": "2024-11-15",
         "total_price": 600.0,
         "status": "confirmed",
         "created_at": "2024-10-01T12:00:00Z"
       }
       ```
     - `400 Bad Request` if validation fails.
     - `401 Unauthorized` if the user is not authenticated or not a guest.
     - `409 Conflict` if the property is already booked for the selected dates.
   - **Validation**:
     - Property ID must be a valid UUID and exist in the database.
     - Guest ID must be a valid UUID and match the authenticated user.
     - Check-in and check-out dates must be valid and the check-out date must be after the check-in date.
     - Total price must be a positive number and match the property's price for the selected dates.
     - Payment method must be one of the allowed values (e.g., "credit_card", "paypal").
   - **Security**:
     - Ensure only authenticated guests can create bookings.
     - Validate JWT token on each request.
     - Implement checks to prevent double-booking of properties.
   - **Performance**:
     - The booking creation endpoint should respond within 600ms under normal load conditions. Database transactions should be optimized for performance.

   ### Feature 3.2: Cancel Booking

   - **Description**: Allows an authenticated guest to cancel an existing booking according to the cancellation policy.
   - **Endpoints**:
     - `DELETE /api/bookings/{booking_id}`
   - **Input Specification**:
     - **Headers**:
       - `Authorization: Bearer <JWT_TOKEN>`
     - **Path Parameters**:
       - `booking_id` (string, required): The unique identifier of the booking to be cancelled.
   - **Response**:
     - `200 OK` on success with cancellation confirmation.
     - **Response Body**:
       ```json
       {
         "message": "Booking cancelled successfully.",
         "booking_id": "b1c2d3e4-f5g6-7890-1234-567890abcdef",
         "status": "cancelled",
         "cancelled_at": "2024-10-02T12:00:00Z"
       }
       ```
     - `400 Bad Request` if validation fails or cancellation policy is not met.
     - `401 Unauthorized` if the user is not authenticated or not the owner of the booking.
     - `404 Not Found` if the booking does not exist.
   - **Validation**:
     - Booking ID must be a valid UUID and exist in the database.
     - Ensure the booking belongs to the authenticated user.
     - Check if the cancellation request meets the property's cancellation policy (e.g., must be made at least 24 hours before check-in).
   - **Security**:
     - Ensure only authenticated guests can cancel their bookings.
     - Validate JWT token on each request.
     - Verify that the authenticated user is the owner of the booking.
   - **Performance**:
     - The booking cancellation endpoint should respond within 500ms under normal load conditions. Database updates should be optimized for performance.

4. Review Management

   ### Feature 4.1: Submit Review

   - **Description**: Allows an authenticated guest to submit a review for a property they have stayed at.
   - **Endpoints**:
     - `POST /api/reviews`
   - **Input Specification**:
     - **Headers**:
       - `Authorization: Bearer <JWT_TOKEN>`
     - **Content-Type**: `application/json`
     - **Request Body**:
       ```json
       {
         "property_id": "p1b2c3d4-e5f6-7890-1234-567890abcdef",
         "guest_id": "guest-uuid-5678",
         "rating": 5,
         "comment": "Amazing stay! The apartment was clean and well-located."
       }
       ```
   - **Response**:
     - `201 Created` on success with review details.
     - **Response Body**:
       ```json
       {
         "review_id": "r1c2d3e4-f5g6-7890-1234-567890abcdef",
         "property_id": "p1b2c3d4-e5f6-7890-1234-567890abcdef",
         "guest_id": "guest-uuid-5678",
         "rating": 5,
         "comment": "Amazing stay! The apartment was clean and well-located.",
         "created_at": "2024-10-01T12:00:00Z"
       }
       ```
     - `400 Bad Request` if validation fails.
     - `401 Unauthorized` if the user is not authenticated or not a guest.
     - `403 Forbidden` if the guest has not completed a stay at the property.
   - **Validation**:
     - Property ID must be a valid UUID and exist in the database.
     - Guest ID must be a valid UUID and match the authenticated user.
     - Rating must be an integer between 1 and 5.
     - Comment must be between 10 and 1000 characters.
     - Ensure the guest has completed a booking at the property before allowing review submission.
   - **Security**:
     - Ensure only authenticated guests can submit reviews.
     - Validate JWT token on each request.
   - **Performance**:
     - The review submission endpoint should respond within 400ms under normal load conditions. Database inserts should be optimized for performance.

5. Payment Integration

   ### Feature 5.1: Process Payment

   - **Description**: Allows an authenticated guest to process payment for a booking using a third-party payment gateway.
   - **Endpoints**:
     - `POST /api/payments`
   - **Input Specification**:
     - **Headers**:
       - `Authorization: Bearer <JWT_TOKEN>`
     - **Content-Type**: `application/json`
     - **Request Body**:
       ```json
       {
         "booking_id": "b1c2d3e4-f5g6-7890-1234-567890abcdef",
         "payment_method": "credit_card",
         "amount": 600.0,
         "currency": "USD",
         "card_details": {
           "card_number": "4111111111111111",
           "expiry_date": "12/25",
           "cvv": "123"
         }
       }
       ```
   - **Response**:
     - `200 OK` on success with payment confirmation.
     - **Response Body**:
       ```json
       {
         "payment_id": "pay-uuid-1234",
         "booking_id": "b1c2d3e4-f5g6-7890-1234-567890abcdef",
         "amount": 600.0,
         "currency": "USD",
         "status": "completed",
         "transaction_id": "txn-uuid-5678",
         "created_at": "2024-10-01T12:00:00Z"
       }
       ```
     - `400 Bad Request` if validation fails.
     - `401 Unauthorized` if the user is not authenticated or not the owner of the booking.
     - `402 Payment Required` if the payment fails.
   - **Validation**:
     - Booking ID must be a valid UUID and exist in the database.
     - Ensure the booking belongs to the authenticated user and is in a state that allows payment (e.g., not already paid).
     - Amount must match the total price of the booking.
     - Payment method must be one of the allowed values (e.g., "credit_card", "paypal").
     - Card details must be valid (e.g., card number format, expiry date in the future, CVV length).
   - **Security**:
     - Ensure only authenticated guests can process payments for their bookings.
     - Validate JWT token on each request.
     - Use HTTPS for all requests.
     - Do not store sensitive card details on the server; use tokenization provided by the payment gateway.
   - **Performance**:
     - The payment processing endpoint should respond within 800ms under normal load conditions. Integration with the payment gateway should be optimized for performance.

   ### Feature 5.2: Handle Payment Webhook

   - **Description**: Handles webhook events from the payment gateway to update booking and payment statuses.
   - **Endpoints**:
     - `POST /api/webhooks/stripe`
   - **Input Specification**:
     - **Headers**:
       - `Content-Type: application/json`
       - `Stripe-Signature: <signature>` (for verifying the webhook)
     - **Request Body**:
       ```json
       {
         "id": "evt_1A2B3C4D5E6F7G8H9I0J",
         "object": "event",
         "type": "payment_intent.succeeded",
         "data": {
           "object": {
             "id": "pi_1A2B3C4D5E6F7G8H9I0J",
             "amount": 60000,
             "currency": "usd",
             "metadata": {
               "booking_id": "b1c2d3e4-f5g6-7890-1234-567890abcdef"
             },
             "status": "succeeded"
           }
         }
       }
       ```
   - **Response**:
     - `200 OK` on successful processing of the webhook.
     - `400 Bad Request` if validation fails.
     - `401 Unauthorized` if the webhook signature is invalid.
   - **Validation**:
     - Verify the webhook signature using the secret provided by the payment gateway.
     - Ensure the event type is one that the system is set up to handle (e.g., "payment_intent.succeeded", "payment_intent.payment_failed").
     - Extract and validate the booking ID from the event metadata.
   - **Security**:
     - Validate the webhook signature to ensure the request is from the payment gateway.
     - Use HTTPS for all requests.
   - **Performance**:
     - The webhook handling endpoint should respond within 300ms under normal load conditions. Database updates should be optimized for performance.
   - **Idempotency**:
     - Ensure that processing the same webhook event multiple times does not result in duplicate updates (e.g., use the event ID to track processed events).
   - **Logging**:
     - Log all webhook events and their processing status for auditing and debugging purposes.

   ### Feature 5.3: Payout to Host

   - **Description**: Automatically processes payouts to hosts after a guest's stay is completed.
   - **Endpoints**:
     - `POST /api/payouts`
   - **Input Specification**:
     - **Headers**:
       - `Authorization: Bearer <JWT_TOKEN>`
     - **Content-Type**: `application/json`
     - **Request Body**:
       ```json
       {
         "host_id": "host-uuid-1234",
         "amount": 540.0,
         "currency": "USD",
         "booking_id": "b1c2d3e4-f5g6-7890-1234-567890abcdef"
       }
       ```
   - **Response**:
     - `200 OK` on success with payout confirmation.
     - **Response Body**:
       ```json
       {
         "payout_id": "payout-uuid-1234",
         "host_id": "host-uuid-1234",
         "amount": 540.0,
         "currency": "USD",
         "status": "completed",
         "transaction_id": "txn-uuid-5678",
         "created_at": "2024-10-01T12:00:00Z"
       }
       ```
     - `400 Bad Request` if validation fails.
     - `401 Unauthorized` if the user is not authenticated or not the owner of the host account.
     - `402 Payment Required` if the payout fails.
   - **Validation**:
     - Host ID must be a valid UUID and exist in the database.
     - Ensure the host ID matches the authenticated user.
     - Amount must be a positive number and match the expected payout for the booking (e.g., total price minus platform fees).
     - Booking ID must be a valid UUID and exist in the database.
     - Ensure the booking is completed and eligible for payout.
   - **Security**:
     - Ensure only authenticated hosts can request payouts.
     - Validate JWT token on each request.
     - Use HTTPS for all requests.
     - Do not store sensitive payment details on the server; use tokenization provided by the payment gateway.
   - **Performance**:
     - The payout processing endpoint should respond within 800ms under normal load conditions. Integration with the payment gateway should be optimized for performance.
   - **Idempotency**:
     - Ensure that processing the same payout request multiple times does not result in duplicate payouts (e.g., use a unique payout ID).
   - **Logging**:
     - Log all payout requests and their processing status for auditing and debugging purposes.

6. Notification System
   ### Feature 6.1: Send Email Notification
   - **Description**: Sends email notifications to users for important events such as booking confirmations, cancellations, and payment receipts.
   - **Endpoints**:
     - `POST /api/notifications/email`
   - **Input Specification**:
     - **Headers**:
       - `Authorization: Bearer <JWT_TOKEN>`
     - **Content-Type**: `application/json`
     - **Request Body**:
       ```json
       {
         "to": "joh.doe@email.com",
         "subject": "Booking Confirmation",
         "body": "Your booking for Cozy Apartment in Downtown has been confirmed."
       }
       ```
   - **Response**:
     - `200 OK` on success with notification details.
     - **Response Body**:
       ```json
       {
         "notification_id": "notif-uuid-1234",
         "to": "john.doe@email.com",
         "subject": "Booking Confirmation",
         "body": "Your booking for Cozy Apartment in Downtown has been confirmed.",
         "status": "sent",
         "sent_at": "2024-10-01T12:00:00Z"
       }
       ```
     - `400 Bad Request` if validation fails.
     - `401 Unauthorized` if the user is not authenticated.
     - `500 Internal Server Error` if the email service fails.
   - **Validation**:
     - Email address must be in a valid format.
     - Subject must be between 5 and 100 characters.
     - Body must be between 10 and 5000 characters.
   - **Security**:
   - Ensure only authenticated users can send notifications.
   - Validate JWT token on each request.
   - Use HTTPS for all requests.
   - **Performance**:
     - The email notification endpoint should respond within 300ms under normal load conditions. Email sending should be handled asynchronously to avoid blocking the request.
   - **Logging**:
     - Log all email notifications and their sending status for auditing and debugging purposes.
   ### Feature 6.2: Send In-App Notification
   - **Description**: Sends in-app notifications to users for important events such as new booking requests, messages, and status updates.
   - **Endpoints**:
     - `POST /api/notifications/in-app`
   - **Input Specification**:
     - **Headers**:
       - `Authorization: Bearer <JWT_TOKEN>`
     - **Content-Type**: `application/json`
     - **Request Body**:
       ```json
       {
         "user_id": "user-uuid-1234",
         "message": "You have a new booking request for your property."
       }
       ```
   - **Response**:
     - `200 OK` on success with notification details.
     - **Response Body**:
       ```json
       {
         "notification_id": "notif-uuid-5678",
         "user_id": "user-uuid-1234",
         "message": "You have a new booking request for your property.",
         "status": "delivered",
         "delivered_at": "2024-10-01T12:00:00Z"
       }
       ```
     - `400 Bad Request` if validation fails.
     - `401 Unauthorized` if the user is not authenticated.
     - `500 Internal Server Error` if the notification service fails.
   - **Validation**:
     - User ID must be a valid UUID and exist in the database.
     - Message must be between 5 and 1000 characters.
   - **Security**:
     - Ensure only authenticated users can send in-app notifications.
     - Validate JWT token on each request.
     - Use HTTPS for all requests.
   - **Performance**:
     - The in-app notification endpoint should respond within 300ms under normal load conditions. Notification sending should be handled asynchronously to avoid blocking the request.
   - **Logging**:
     - Log all in-app notifications and their sending status for auditing and debugging purposes.
