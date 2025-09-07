# MinIO Multi-User Demo

This project demonstrates how to set up MinIO with multiple users and policies for different microservices using Docker Compose.

## Quick Start

1. Clone the repository:
```bash
git clone https://github.com/Agile-Software-Engineering-25/demo-minio-multiuser.git
cd demo-minio-multiuser
```

2. Start the MinIO setup:
```bash
docker-compose up -d
```

3. Wait for the setup to complete (check logs):
```bash
docker-compose logs minio-setup
```

4. Access MinIO Console: http://localhost:9001
   - Username: `admin`
   - Password: `adminpassword`

## Architecture

### Users Created

| User | Password | Bucket Access | Description |
|------|----------|---------------|-------------|
| `user-service-user` | `userServicePassword123` | `user-service-bucket` (full) | User management service |
| `auth-service-user` | `authServicePassword123` | `auth-service-bucket` (full), `shared-bucket` (read) | Authentication service |
| `file-service-user` | `fileServicePassword123` | `file-service-bucket` (full), `shared-bucket` (full) | File management service |
| `notification-service-user` | `notificationServicePassword123` | `notification-service-bucket` (full), all others (read) | Notification service |
| `readonly-user` | `readonlyPassword123` | All buckets (read-only) | Monitoring/debugging |

### Buckets Created

- `user-service-bucket` - User profile data, avatars
- `auth-service-bucket` - Authentication tokens, certificates
- `file-service-bucket` - General file uploads
- `notification-service-bucket` - Email templates, notification assets
- `shared-bucket` - Shared resources between services

### Policies

Each service has its own IAM policy with appropriate permissions:

- **User Service**: Full access to `user-service-bucket` only
- **Auth Service**: Full access to `auth-service-bucket`, read access to `shared-bucket`
- **File Service**: Full access to `file-service-bucket` and `shared-bucket`
- **Notification Service**: Full access to `notification-service-bucket`, read access to all other buckets
- **Read-only**: Read access to all buckets for monitoring

## Using in Your Applications

### Environment Variables

Set these in your microservice environment:

```env
# User Service
MINIO_ENDPOINT=http://localhost:9000
MINIO_ACCESS_KEY=user-service-user
MINIO_SECRET_KEY=userServicePassword123
MINIO_BUCKET=user-service-bucket

# Auth Service
MINIO_ENDPOINT=http://localhost:9000
MINIO_ACCESS_KEY=auth-service-user
MINIO_SECRET_KEY=authServicePassword123
MINIO_BUCKET=auth-service-bucket

# File Service
MINIO_ENDPOINT=http://localhost:9000
MINIO_ACCESS_KEY=file-service-user
MINIO_SECRET_KEY=fileServicePassword123
MINIO_BUCKET=file-service-bucket

# Notification Service
MINIO_ENDPOINT=http://localhost:9000
MINIO_ACCESS_KEY=notification-service-user
MINIO_SECRET_KEY=notificationServicePassword123
MINIO_BUCKET=notification-service-bucket
```

### Example Code (Node.js with AWS SDK)

```javascript
const AWS = require('aws-sdk');

const s3 = new AWS.S3({
  endpoint: process.env.MINIO_ENDPOINT,
  accessKeyId: process.env.MINIO_ACCESS_KEY,
  secretAccessKey: process.env.MINIO_SECRET_KEY,
  s3ForcePathStyle: true,
  signatureVersion: 'v4'
});

// Upload a file
const uploadFile = async (filename, buffer) => {
  return s3.upload({
    Bucket: process.env.MINIO_BUCKET,
    Key: filename,
    Body: buffer
  }).promise();
};
```

## Customization

### Adding New Services

1. Edit `scripts/setup-minio.sh` to add new users, buckets, and policies
2. Add corresponding policy files in the `policies/` directory
3. Restart the setup: `docker-compose down && docker-compose up -d`

### Modifying Policies

1. Edit the policy files in `policies/` directory
2. Restart the setup container: `docker-compose restart minio-setup`

## Troubleshooting

- **Setup fails**: Check logs with `docker-compose logs minio-setup`
- **Access denied**: Verify the user has the correct policy assigned
- **Connection refused**: Ensure MinIO is running on port 9000

## Security Notes

- Change default passwords in production
- Use HTTPS in production environments
- Consider using external secret management
- Regularly audit user permissions
