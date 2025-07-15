# Story Map - Release 1: Basic Features

## Customer

- Purchase deals
  - Search deals
    - By featured deals ^
    - By category
    - By latest deals
    - By merchant
  - View a deal ^
  - Buy the deal ^
  - Save a deal for later
- Manage purchased deals
  - View saved deals
  - View purchase history
- Manage account
  - Self-sign up
    - With username and password
      - Successful sign up
      - Account already exists
    - With SSO
  - Sign in
    - With username and password
    - With SSO
  - Sign out
  - Update account
    - Change password
  - Delete account

## Merchant

- Get information about becoming a merchant
  - View a landing page with
    - Benefits
    - How to apply
    - FAQ
- Manage account
  - Complete verification for a pre-verified merchant account
  - Self-sign up for a pre-verified merchant account
    - Successful sign up
    - Account already exists
  - Sign in
    - Stay signed in for 24 hours
      - Need to make the session expiry 24 hours in cognito/amplify. default is 1 hour.
  - Sign out
  - Update account
    - Change password
  - Delete account
- Manage deals
  - Add a deal
  - List all deals
  - Suspend a deal
  - Update a deal
  - Delete a deal
  - View a deal
- Receive training
- Receive support

## Platform Admin

- Manage account
  - Rule: user cannot register for this type of account. It can only be created by a super user
  - Sign in
  - Sign out
  - Update account
    - Change password
  - Delete account
- Manage merchants
  - Process merchant account application ^
  - View merchants
  - Suspend merchants
- Manage deal listings
  - Approve deals
  - View deals
  - Suspend deals

## Auth

- A user who needs to verify their email can request a resend of the verification code
  - Successful resend
  - Failed resend

## General

- Protected routes is implemented for all users
