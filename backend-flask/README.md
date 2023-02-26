# Install python version

```
pyenv install 3.10.9
```

# Set your python version

```
pyenv global 3.10.9
```

# Create virual environment

```
python -m venv venv
```

# Activate environment

```
source venv/bin/activate
```

# Install Flask

```
pip install flask
```

# Install deps

```
cd backend-flask
pip3 install -r requirements.txt
```

# Run Backend

```
cd backend-flask
export BACKEND_URL=*
export FRONTEND_URL=*
python3 -m flask run --host=0.0.0.0 --port=4567
```
