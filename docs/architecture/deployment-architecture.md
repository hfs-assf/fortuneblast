# Deployment Architecture

## 1. Overview

The system uses a single Git repository with
independent deployment pipelines.

The mobile application and backend are deployed
separately.

## 2. Source Code

GitHub repository:

laundry/

Applications:

- apps/mobile
- apps/api

Shared package:

- packages/shared

## 3. Mobile Deployment

Technology:

- Expo
- React Native
- TypeScript
- EAS Build

Deployment process:

1. Developer pushes mobile changes.
2. GitHub Actions runs validation and tests.
3. EAS Build generates the Android application.
4. The application is submitted to Google Play Store.
5. Users install the application from Google Play.

## 4. Backend Deployment

Technology:

- NestJS
- Node.js
- Docker

Initial hosting:

- DigitalOcean Droplet

Deployment process:

1. Developer pushes backend changes.
2. GitHub Actions runs tests.
3. Docker image is built.
4. Image is pushed to a container registry.
5. DigitalOcean server pulls the new image.
6. Backend service is restarted.
7. Database migrations are executed through
   a controlled release process.

## 5. Database Deployment

Technology:

- PostgreSQL
- Prisma ORM

Initial development:

- PostgreSQL runs using Docker Compose.

Production:

- PostgreSQL runs in private cloud infrastructure.
- The database is not publicly accessible.
- Only the backend can connect to the database.

Managed PostgreSQL may be adopted when appropriate.

## 6. Network Communication

Mobile application communicates with the backend
through HTTPS REST API.

Example:

https://api.example.com/api/v1

The mobile application must never connect directly
to PostgreSQL.

## 7. Security

- HTTPS for all production API communication.
- JWT authentication.
- Backend authorization.
- Secure environment variables.
- Database network restrictions.
- Automated database backups.
- Audit logging.
