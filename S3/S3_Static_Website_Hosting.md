# 🌐 Hosting a Static Website on Amazon S3

This guide walks you through creating an Amazon S3 bucket, configuring it for static website hosting, setting up permissions, and accessing your hosted site.

---

## 🗺️ Architecture Overview

```mermaid
graph TD
  A[Create S3 Bucket] --> B[Enable Static Website Hosting]
  B --> C[Upload Website Files]
  C --> D[Set Bucket Policy for Public Access]
  D --> E[Access Website via S3 Endpoint]
```

---

## 🪣 Step 1: Create an S3 Bucket

> ✅ Ensure bucket name is globally unique and regionally appropriate.

![bucket is created named khushalsavalakha](https://github.com/user-attachments/assets/358b0ee0-1137-4f21-a8a1-7e1a109e4245)

---

## ⚙️ Step 2: Enable Static Website Hosting

> Navigate to the **Properties** tab of the bucket and scroll to the **Static website hosting** section.

![static website hosting](https://github.com/user-attachments/assets/445daecb-a451-4939-b711-3ebf945045c0)

Click on **Edit**.

![Go to Static Website Hosting](https://github.com/user-attachments/assets/404dbdcd-c83f-49c6-9594-8c74dcfb8f87)

Add the **Index document** and optional **Error document** (e.g., `index.html`, `error.html`).

![Add index page and error page](https://github.com/user-attachments/assets/d18fb4d7-cee8-4503-8768-220aee595159)

Click **Save Changes**.

![Click on save Changes](https://github.com/user-attachments/assets/5bfc6c83-6f11-4d39-ba7c-1cef76ea3425)

---

## 📂 Step 3: Upload Static Files

> Upload your website files such as `index.html`, `error.html`, CSS, images, etc.

![Add Static Website data](https://github.com/user-attachments/assets/f21f9ea0-05f6-4374-97fa-efaaa13d0fa3)

---

## 🔐 Step 4: Configure Bucket Policy for Public Access

> To allow public access, edit the bucket policy as shown below.

![edit bucket policy](https://github.com/user-attachments/assets/b5300f3a-ab36-4e5f-aeab-55225d691956)

### ✅ Bucket Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadAccess",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::khushalgolfclub/*"
    }
  ]
}
```

![show bucket policy](https://github.com/user-attachments/assets/e4772c29-4b01-4101-9f3b-fc95a7f71850)

Click **Save Changes**.

![Successfully Initiated bucket policy](https://github.com/user-attachments/assets/2b4ccbae-da4d-4a97-a85e-1a8ac6bce4fe)

---

## 🌐 Step 5: Access Your Hosted Website

> Find the website endpoint URL under the static website hosting section.

![Now you can see our static website url](https://github.com/user-attachments/assets/550c6cfc-ea0f-4d13-a01a-5182b05eac9b)

Click on the URL to view your live website:

![Click on url and we can see website coffe shop page](https://github.com/user-attachments/assets/f2db252e-c91e-4c83-ac68-55822277a11e)

Try entering an invalid path to test your error page:

![we can see error page if wrong url is entered](https://github.com/user-attachments/assets/bfd3b7e8-7956-49a9-8472-4acd212292c3)

---

## ✅ Conclusion

You have successfully:

* Created and configured an Amazon S3 bucket.
* Enabled static website hosting.
* Configured a bucket policy for public read access.
* Hosted and tested a static website on S3.

