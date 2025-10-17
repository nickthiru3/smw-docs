Story: Merchant creates a deal

As a merchant
I want to be able to create a deal
So that I can sell my products and/or services

---

Rule: Merchant must be signed-in to create a deal

Scenario: Merchant successfully creates a deal

Scenario: Merchant is not able to go to "Create a deal" page without signing-in

---

BACKEND:

API Gateway

    HTTP: https://merchant.superdeals.com

      POST /merchant/deals

        Body

          {
            merchantId: string; required; ID of merchant who created the deal,
            title: string; required; Title of deal,
            originalPrice: number, required; Price of deal,
            discount: number, required; Deal discount (to calculate final deal price),
            logo: PNG/JPEG/SVG file,
            category: "Food & Drink" | "Bathroom" | "Jewelery" | "Sports" | "Tech" | "Auto" | "Entertainment" | "Travel"; required; The category this deal falls under,
            expiration: date; required; The expiration date of the deal,
          }

        Responses

          200

            {
              title: string; required; Title of deal,
              originalPrice: number, required; Price of deal,
              discount: number, required; Deal discount (to calculate final deal price),
              logoUrl: string; required; S3 Bucket URL of deal's logo,
              category: "Food & Drink" | "Bathroom" | "Jewelery" | "Sports" | "Tech" | "Auto" | "Entertainment" | "Travel"; required; The category this product falls under,
              expiration: date; required; The expiration date of the deal,
            }

S3

    Bucket

      Logo Key

DynamoDB

    Entity

      Deal

        {
          PK: DEAL#<DealId>,
          SK: DEAL#<DealId>,
          EntityType: string; required, Entity type,
          Id: KSUID,
          Title: string; required; Title of deal,
          OriginalPrice: number, required; Price of deal,
          Discount: number, required; Deal discount (to calculate final deal price),
          LogoUrl: string; required; S3 Bucket URL of deal's logo,
          Rating: number; optional; Overall rating of deal,
          Category: "Food & Drink" | "Bathroom" | "Jewelery" | "Sports" | "Tech" | "Auto" | "Entertainment" | "Travel"; required; The category this deal falls under,
          Expiration: date; required; The expiration date of the deal,
          MerchantId: string; required; ID of merchant who created the deal,
        }

Lambda

    add-deal

FRONTEND:

Route

    /merchant/deals/add

      Page

        "Add a deal" form

      Page Server

        Submit form
