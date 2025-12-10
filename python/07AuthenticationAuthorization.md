# Authentication & Authorization

## Table of Content

- [Authentication \& Authorization](#authentication--authorization)
  - [Table of Content](#table-of-content)
  - [JWT](#jwt)

## JWT

How to implement JWT in python with Entra ID.

Step 1) get registration details:

```python
# config.py
import os

# MS Entra App Registration Details
CLIENT_ID = os.getenv('AZURE_CLIENT_ID', 'your-client-id')
CLIENT_SECRET = os.getenv('AZURE_CLIENT_SECRET', 'your-client-secret')
TENANT_ID = os.getenv('AZURE_TENANT_ID', 'your-tenant-id')
SCOPE = ['https://graph.microsoft.com/.default']

# MS Entra Endpoints
AUTHORITY = f'https://login.microsoftonline.com/{TENANT_ID}'
```

Step 2) Create Auth client

```python
# jwt_client.py
import msal
import requests
import jwt
from typing import Dict, Optional
from config import CLIENT_ID, CLIENT_SECRET, TENANT_ID, AUTHORITY, SCOPE

class MSEntraJWTClient:
    def __init__(self):
        self.app = msal.ConfidentialClientApplication(
            CLIENT_ID,
            authority=AUTHORITY,
            client_credential=CLIENT_SECRET,
        )
        self.token_cache = {}
    
    def get_access_token(self) -> Optional[str]:
        """
        Get access token from MS Entra using client credentials flow
        """
        try:
            # Try to get token from cache first
            result = self.app.acquire_token_silent(SCOPE, account=None)
            
            if not result:
                # Get new token
                result = self.app.acquire_token_for_client(scopes=SCOPE)
            
            if "access_token" in result:
                return result["access_token"]
            else:
                print(f"Error getting token: {result.get('error_description')}")
                return None
                
        except Exception as e:
            print(f"Exception getting token: {str(e)}")
            return None
    
    def get_user_token(self, username: str, password: str) -> Optional[str]:
        """
        Get user token using Resource Owner Password Credentials (ROPC) flow
        Note: ROPC is not recommended for production use
        """
        try:
            result = self.app.acquire_token_by_username_password(
                username=username,
                password=password,
                scopes=SCOPE
            )
            
            if "access_token" in result:
                return result["access_token"]
            else:
                print(f"Error getting user token: {result.get('error_description')}")
                return None
                
        except Exception as e:
            print(f"Exception getting user token: {str(e)}")
            return None
```

Step 3) validate tokens:

```python
# jwt_validator.py
import jwt
import requests
from typing import Dict, Optional
from cryptography.hazmat.primitives import serialization
from config import TENANT_ID, CLIENT_ID

class JWTValidator:
    def __init__(self):
        self.jwks_url = f'https://login.microsoftonline.com/{TENANT_ID}/discovery/v2.0/keys'
        self.issuer = f'https://login.microsoftonline.com/{TENANT_ID}/v2.0'
        self.audience = CLIENT_ID
        self._jwks_cache = None
    
    def _get_jwks(self) -> Dict:
        """Get JSON Web Key Set from MS Entra"""
        if not self._jwks_cache:
            response = requests.get(self.jwks_url)
            response.raise_for_status()
            self._jwks_cache = response.json()
        return self._jwks_cache
    
    def _get_signing_key(self, token_header: Dict) -> str:
        """Get the signing key for token validation"""
        jwks = self._get_jwks()
        
        for key in jwks['keys']:
            if key['kid'] == token_header['kid']:
                # Convert JWK to PEM format
                key_obj = jwt.algorithms.RSAAlgorithm.from_jwk(key)
                return key_obj.public_key().public_bytes(
                    encoding=serialization.Encoding.PEM,
                    format=serialization.PublicFormat.SubjectPublicKeyInfo
                )
        
        raise jwt.InvalidKeyError(f"Unable to find key with kid: {token_header['kid']}")
    
    def validate_token(self, token: str) -> Optional[Dict]:
        """
        Validate JWT token and extract claims
        """
        try:
            # Decode token header to get key ID
            unverified_header = jwt.get_unverified_header(token)
            
            # Get signing key
            signing_key = self._get_signing_key(unverified_header)
            
            # Validate and decode token
            decoded_token = jwt.decode(
                token,
                signing_key,
                algorithms=['RS256'],
                audience=self.audience,
                issuer=self.issuer,
                options={"verify_exp": True}
            )
            
            return decoded_token
            
        except jwt.ExpiredSignatureError:
            print("Token has expired")
            return None
        except jwt.InvalidAudienceError:
            print("Invalid audience")
            return None
        except jwt.InvalidIssuerError:
            print("Invalid issuer")
            return None
        except jwt.InvalidSignatureError:
            print("Invalid signature")
            return None
        except Exception as e:
            print(f"Token validation error: {str(e)}")
            return None
    
    def extract_user_claims(self, token: str) -> Optional[Dict]:
        """
        Extract user-specific claims from JWT token
        """
        claims = self.validate_token(token)
        if not claims:
            return None
        
        user_claims = {
            'user_id': claims.get('sub'),
            'username': claims.get('preferred_username'),
            'name': claims.get('name'),
            'email': claims.get('email'),
            'roles': claims.get('roles', []),
            'groups': claims.get('groups', []),
            'tenant_id': claims.get('tid'),
            'app_id': claims.get('aud'),
            'issued_at': claims.get('iat'),
            'expires_at': claims.get('exp')
        }
        
        return user_claims
```

Step 4) integrate in application

```python
# app.py
from flask import Flask, request, jsonify
from jwt_client import MSEntraJWTClient
from jwt_validator import JWTValidator
from functools import wraps

app = Flask(__name__)
jwt_client = MSEntraJWTClient()
jwt_validator = JWTValidator()

def require_auth(f):
    """Decorator to require valid JWT token"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        auth_header = request.headers.get('Authorization')
        
        if not auth_header or not auth_header.startswith('Bearer '):
            return jsonify({'error': 'Missing or invalid Authorization header'}), 401
        
        token = auth_header.split(' ')[1]
        claims = jwt_validator.validate_token(token)
        
        if not claims:
            return jsonify({'error': 'Invalid token'}), 401
        
        # Add claims to request context
        request.user_claims = claims
        return f(*args, **kwargs)
    
    return decorated_function

def require_role(required_role):
    """Decorator to require specific role"""
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            user_roles = request.user_claims.get('roles', [])
            
            if required_role not in user_roles:
                return jsonify({'error': 'Insufficient permissions'}), 403
            
            return f(*args, **kwargs)
        return decorated_function
    return decorator

@app.route('/login', methods=['POST'])
def login():
    """Login endpoint to get JWT token"""
    data = request.get_json()
    username = data.get('username')
    password = data.get('password')
    
    if not username or not password:
        return jsonify({'error': 'Username and password required'}), 400
    
    token = jwt_client.get_user_token(username, password)
    
    if token:
        user_claims = jwt_validator.extract_user_claims(token)
        return jsonify({
            'access_token': token,
            'user_claims': user_claims
        })
    else:
        return jsonify({'error': 'Authentication failed'}), 401

@app.route('/profile', methods=['GET'])
@require_auth
def get_profile():
    """Get user profile from JWT claims"""
    user_claims = jwt_validator.extract_user_claims(
        request.headers.get('Authorization').split(' ')[1]
    )
    
    return jsonify({
        'profile': user_claims
    })

@app.route('/admin', methods=['GET'])
@require_auth
@require_role('Admin')
def admin_endpoint():
    """Admin-only endpoint"""
    return jsonify({
        'message': 'Welcome to admin area',
        'user': request.user_claims.get('preferred_username')
    })

if __name__ == '__main__':
    app.run(debug=True)
```

Step 5) Example with API

```python
# example_usage.py
from jwt_client import MSEntraJWTClient
from jwt_validator import JWTValidator

def main():
    # Initialize clients
    jwt_client = MSEntraJWTClient()
    jwt_validator = JWTValidator()
    
    # Get application token (client credentials)
    app_token = jwt_client.get_access_token()
    if app_token:
        print("Application Token acquired successfully!")
        
        # Validate and extract claims
        claims = jwt_validator.validate_token(app_token)
        if claims:
            print(f"Token valid. App ID: {claims.get('aud')}")
            print(f"Expires at: {claims.get('exp')}")
        else:
            print("Token validation failed")
    
    # Example: Get user token (for demo - use proper auth flow in production)
    user_token = jwt_client.get_user_token('user@domain.com', 'password')
    if user_token:
        print("User Token acquired successfully!")
        
        # Extract user claims
        user_claims = jwt_validator.extract_user_claims(user_token)
        if user_claims:
            print(f"User: {user_claims.get('name')}")
            print(f"Email: {user_claims.get('email')}")
            print(f"Roles: {user_claims.get('roles')}")

if __name__ == '__main__':
    main()
```
