from flask import Flask, request, redirect, url_for, render_template, flash
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///users.db'
app.config['SECRET_KEY'] = 'your_secret_key'
db = SQLAlchemy(app)

# User model
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    surname = db.Column(db.String(100), nullable=False)
    id_number = db.Column(db.String(20), unique=True, nullable=False)
    phone_number = db.Column(db.String(15), unique=True, nullable=False)
    account_number = db.Column(db.String(20), unique=True, nullable=False)

# Create the database
with app.app_context():
    db.create_all()

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/signup', methods=['GET', 'POST'])
def signup():
    if request.method == 'POST':
        name = request.form['name']
        surname = request.form['surname']
        id_number = request.form['id_number']
        phone_number = request.form['phone_number']
        account_number = request.form['account_number']

        new_user = User(name=name, surname=surname, id_number=id_number, phone_number=phone_number, account_number=account_number)

        db.session.add(new_user)
        db.session.commit()

        # Here you would send a confirmation SMS
        flash('Sign up successful! A confirmation SMS has been sent.', 'success')
        return redirect(url_for('index'))

    return render_template('signup.html')

@app.route('/signin', methods=['GET', 'POST'])
def signin():
    if request.method == 'POST':
        account_number = request.form['account_number']
        phone_number = request.form['phone_number']
        id_number = request.form['id_number']

        user = User.query.filter_by(account_number=account_number, phone_number=phone_number, id_number=id_number).first()

        if user:
            flash('Sign in successful!', 'success')
            return redirect(url_for('index'))
        else:
            flash('Sign in failed! Please check your credentials.', 'danger')

    return render_template('signin.html')

if __name__ == '__main__':
    app.run(debug=True)
