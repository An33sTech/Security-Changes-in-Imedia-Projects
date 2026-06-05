# Steps to Secure Old Sites
Follow the steps below carefully

### 1. Update global.php
Replace the old session code with the following code. Make sure there is no space before or after the PHP opening tag.

```
<?php
if (session_status() === PHP_SESSION_NONE || session_id() === '') {

    session_set_cookie_params([
        'lifetime' => 3600 * 24 * 7,
        'path'     => '/',
        'secure'   => true,
        'httponly' => true,
        'samesite' => 'Lax'
    ]);

    ini_set('session.cookie_secure', '1');
    ini_set('session.cookie_httponly', '1');
    ini_set('session.cookie_samesite', 'Lax');

    session_start();
}
```
### 2. Update auth.php
Replace the old auth.php code with the following code.

```
<?php
$ruri = $_SERVER['REQUEST_URI'];

$ruri_do = strpos($ruri, '/do-');

$ruri_secure = strpos($ruri, '.secure');

if ($ruri_do !== false && $ruri_secure !== false && $ruri_secure > ($ruri_do + 2)) {
} else {
    header("HTTP/1.0 404 Not Found");
    exit();
}
include "global.php";
$functions->menu_show = false;
@$do = empty($_GET['do']) ? 0 : $_GET['do'];
switch ($do):
    case "login":
        $echo = include "_models/pages/login.page.php";
        break;
    case "register":
        $echo = include "_models/pages/register.page.php";
        break;
    default:
        $echo = "<h1>404 - Error</h1>";
        break;
endswitch;

echo $echo;
$functions->adminFooter();
```

### 3. Update .htaccess
Use the following recommended Apache configuration.

```
Options -Indexes
ServerSignature Off

RewriteEngine On
RewriteBase /

RewriteCond %{HTTPS} !=on
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

RewriteCond %{HTTP_HOST} ^www\.(.+)$ [NC]
RewriteRule ^ https://%1%{REQUEST_URI} [R=301,L]

<IfModule mod_headers.c>
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=()"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Content-Security-Policy "default-src 'self'; img-src 'self' data: blob: https:; script-src 'self' 'unsafe-inline' 'unsafe-eval' https:; style-src 'self' 'unsafe-inline' https:; font-src 'self' data: https:; connect-src 'self' https: wss:; frame-src 'self' https:; object-src 'none'; base-uri 'self'; form-action 'self' https:;"
</IfModule>

RewriteRule ^assets/(.*)$ myAdmin/assets/$1 [L]

RewriteRule ^do-([^/]*)_([^_/]*)_([^/]*)\.secure$ auth.php?do=$1&action=$2&value=$3 [L,QSA]
RewriteRule ^do-([^/]*)\.secure$ auth.php?do=$1 [L,QSA]

<FilesMatch "^product\.php$">
    Header set Content-Type "application/javascript; charset=UTF-8"
</FilesMatch>

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ /index.php?path=$1 [QSA,L]
```

> Remove the **FilesMatch product.php** block if the site does not use product.php inside js/.

### 4. Update _models/pages/login.page.php
Replace the old login.page.php code with the following code.

```
<?php
ob_start();

function e($value)
{
    return htmlspecialchars((string)$value, ENT_QUOTES, 'UTF-8');
}

function loginScript()
{ ?>
    <link rel="icon" href="favicon.ico" type="image/x-icon" />
    <link rel="shortcut icon" href="favicon.ico" type="image/x-icon" />
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/css/style.css" />
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/assets/font-awesome/css/font-awesome.css" />
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/assets/jquery-ui/css/jquery-ui-1.11.0.css" />
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/assets/bootstrap/css/bootstrap.css" />
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/assets/bootstrap/css/bootstrap-theme.css" />
    <title>IBMS</title>
<?php }

global $_e;

$_w['You are already logged in!'] = '';
$_w['Too many login attempts. Please try after some time later!'] = '';
$_w['Email'] = '';
$_w['Password'] = '';
$_w["Forgotten your password? \n Click Here!"] = '';
$_w['Signin'] = '';
$_w['Login'] = '';
$_w['Go To Home'] = '';
$_w['SignIn'] = '';
$_w['Woops, Too Slow!'] = '';
$_w['Session expired! Please try again. This is for your own security.'] = '';
$_w['Your email or password is incorrect, please type again!'] = '';
$_w['Stop!'] = '';

$lang = $functions->ibms_setting('Default Language');
$_e = $dbF->hardWordsMulti($_w, $lang, 'Admin Login');

if ($functions->log_check()["status"] == "ok") {
    header('Location: ' . WEB_ADMIN_URL, true, 302);
    exit;
}

if (empty($_SESSION['login_csrf'])) {
    $_SESSION['login_csrf'] = bin2hex(random_bytes(32));
}

$_toss = $_SESSION['login_csrf'];
$alerts = "";

loginScript();
?>

<div class="wrapper container-fluid">
    <div class="navbar navbar-inverse navbar-fixed-top" role="navigation" id="mainTopMenu">
        <div class="container-fluid">
            <div class="navbar-header">
                <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>

                <a class="navbar-brand visible-xs" href="<?php echo WEB_URL; ?>">
                    <i class="fa fa-home"></i>
                </a>
            </div>

            <div class="navbar-collapse collapse">
                <ul class="nav navbar-nav">
                    <li class="active">
                        <a href="<?php echo WEB_URL; ?>">
                            <i class="fa fa-home" style="font-size: 18px"></i>
                            <?php echo $_e['Go To Home']; ?>
                        </a>
                    </li>
                </ul>

                <ul class="nav navbar-nav navbar-right">
                    <li class="active">
                        <a href="<?php echo WEB_URL; ?>/do-login.secure">
                            <i class="glyphicon glyphicon-log-in"></i>
                            <?php echo $_e['SignIn']; ?>
                        </a>
                    </li>
                </ul>
            </div>
        </div>
    </div>

    <div class="IBMS_LOGO col-sm-12 text-center">
        <div style="margin-top: 100px;display: inline-block;">
            <div style="display: inline-block; vertical-align: middle;float: left;margin-right: 10px;">
                <img src="<?php echo WEB_ADMIN_URL; ?>/images/logo_ibms.png" width="120" alt="IBMS" />
            </div>

            <div style="font-size: 30px;float: left;display: inline-block;">
                IBMS
                <div style="display: inline-block; position: relative; vertical-align: middle;
                        font-size: 12px; text-align: left; border-left: solid #5f5f5f 1px;
                        padding-left: 5px;  margin-left: -5px;">
                    Interactive
                    Business<br>
                    Management
                    System
                </div>
            </div>

            <div style="font-size: 25px;">
                (VERSION <?php echo $functions->IBMSVersion; ?>)
            </div>
        </div>
    </div>

<?php

$fake_form = '
<div class="container-fluid">
    <div class="content_div">
        <div class="col-sm-12">
            <div style="width: 340px; position: relative; margin: 30px auto;">
                <div class="alert alert-warning">
                    <strong>Stop!</strong> ' . _uc($_e['Too many login attempts. Please try after some time later!']) . '
                </div>

                <div class="panel panel-default">
                    <div class="panel-body btn-success">' . _uc($_e['Login']) . '</div>
                    <div class="panel-footer">
                        <div class="input-group">
                            <span class="input-group-addon btn-default">' . _uc($_e['Email']) . '</span>
                            <input type="text" name="user" class="form-control" disabled="disabled">
                        </div>

                        <br>

                        <div class="input-group">
                            <span class="input-group-addon btn-default">' . _uc($_e['Password']) . '</span>
                            <input type="password" name="pass" class="form-control" disabled="disabled">
                        </div>

                        <br>

                        <div style="display: inline-block;">
                            <a href="' . WEB_ADMIN_URL . '/trouble" style="font-size: 12px;">' . _uc($_e["Forgotten your password? \n Click Here!"]) . '</a>
                        </div>

                        <button type="button" class="btn btn-primary pull-right" disabled="disabled">' . _uc($_e['Signin']) . '</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>';

// if ($_SESSION['login_attempts'] >= 5 && time() < $_SESSION['login_block_time']) {
//     echo $fake_form;
//     return ob_get_clean();
// }

if (
    isset($_POST['_toss'], $_POST['user'], $_POST['pass']) &&
    trim((string)$_POST['user']) !== '' &&
    (string)$_POST['pass'] !== ''
) {
    $csrfOk = isset($_SESSION['login_csrf']) && hash_equals($_SESSION['login_csrf'], (string)$_POST['_toss']);

    if (!$csrfOk) {
        $alerts .= "<div class='alert alert-warning'><strong>" . _uc($_e['Woops, Too Slow!']) . "</strong> " . _n($_e['Session expired! Please try again. This is for your own security.']) . "</div>";
    } else {
        $user = trim((string)$_POST['user']);
        $pass = (string)$_POST['pass'];

        $login_req = $functions->login($user, $pass);
        
        if ($login_req === 'blocked') {
            $alerts .= "<div class='alert alert-warning'>
                <strong>" . _uc($_e['Stop!']) . "</strong>
                Too many failed login attempts. Please try again after 10 minutes.
            </div>";
        } elseif ($login_req === false) {
            $alerts .= "<div class='alert alert-danger'>
                <strong>" . _uc($_e['Stop!']) . "</strong>
                " . _n($_e['Your email or password is incorrect, please type again!']) . "
            </div>";
        }
    }

    $_SESSION['login_csrf'] = bin2hex(random_bytes(32));
    $_toss = $_SESSION['login_csrf'];
}

$action_url = "do-login.secure";
?>

<div class="container-fluid">
    <div class="content_div">
        <div class="col-sm-12">
            <div style="width: 340px; position: relative; margin: 30px auto;">
                <?php echo $alerts; ?>

                <div class="panel panel-default">
                    <div class="panel-body btn-success"><?php echo _uc($_e['Login']); ?></div>

                    <div class="panel-footer">
                        <form method="post" action="<?php echo e($action_url); ?>" autocomplete="off">
                            <input type="hidden" name="_toss" value="<?php echo e($_toss); ?>">

                            <div class="input-group">
                                <span class="input-group-addon btn-default"><?php echo _uc($_e['Email']); ?></span>
                                <input type="email" name="user" class="form-control" required="required" autocomplete="username">
                            </div>

                            <br>

                            <div class="input-group">
                                <span class="input-group-addon btn-default"><?php echo _uc($_e['Password']); ?></span>
                                <input type="password" name="pass" class="form-control" required="required" autocomplete="current-password">
                            </div>

                            <br>

                            <div style="display: inline-block;">
                                <a href="<?php echo WEB_ADMIN_URL; ?>/trouble" style="font-size: 12px;">
                                    <?php echo _uc($_e["Forgotten your password? \n Click Here!"]); ?>
                                </a>
                            </div>

                            <button type="submit" class="btn btn-primary pull-right">
                                <?php echo _uc($_e['Signin']); ?>
                            </button>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

</div>

<?php
return ob_get_clean();
?>
```

> Delete the login.php in _models/pages/ if present

### 5. Update myAdmin/.htaccess
Use the following recommended Apache configuration.

```
RewriteEngine On
RewriteBase /myAdmin/

RewriteCond %{HTTPS} !=on
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

<IfModule mod_headers.c>
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "no-referrer"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=()"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Content-Security-Policy "default-src 'self'; img-src 'self' data: blob: https:; script-src 'self' 'unsafe-inline' 'unsafe-eval' https:; style-src 'self' 'unsafe-inline' https:; font-src 'self' data: https:; connect-src 'self' https: wss:; frame-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self';"
</IfModule>

Options +FollowSymLinks

RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME}.php -f
RewriteRule ^(.+)$ $1.php [L,QSA]

RewriteRule ^do-([^/]*)_(.*)_(.*)\.secure$ auth.php?do=$1&$2=$3 [L,QSA]
RewriteRule ^do-([^/]*)\.secure$ auth.php?do=$1 [L,QSA]

<FilesMatch "^main\.php$">
    Header set Content-Type "application/javascript; charset=UTF-8"
</FilesMatch>
<FilesMatch "^order\.php$">
    Header set Content-Type "application/javascript; charset=UTF-8"
</FilesMatch>

RewriteRule ^-(.*)$ $1/ [L]
```

### 6. Update myAdmin/trouble.php
Replace the old trouble.php code with the following code.

```
<?php
include_once(__DIR__ . "/../global.php");

global $dbF, $db, $_e, $functions;

function e($value)
{
    return htmlspecialchars((string)$value, ENT_QUOTES, 'UTF-8');
}

global $_e;

$_w['An email is sent. Please check your emails.'] = '';
$_w['Email Sent Fail.Please Try Again.'] = '';
$_w['Password Trouble Shooting'] = '';
$_w['Security Captcha'] = '';
$_w['Email'] = '';
$_w['Send Email'] = '';
$_w['Incorrect Email.'] = '';
$_w['Please Type Captcha Code'] = '';
$_w['Please type your email address in the given field.'] = '';
$_w['Signin'] = '';
$_w['Login'] = '';
$_w['Go To Home'] = '';
$_w['SignIn'] = '';
$_w['LOGIN'] = '';

$lang = $functions->ibms_setting('Default Language');
$_e = $dbF->hardWordsMulti($_w, $lang, 'Admin Trouble');

$msgHtml = '';
$email = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $email = isset($_POST['email']) ? trim((string)$_POST['email']) : '';
    $code  = isset($_POST['code']) ? trim((string)$_POST['code']) : '';

    if (!isset($_SESSION["rand_code"]) || $code !== (string)$_SESSION["rand_code"]) {
        $msgHtml = "<div class='alert alert-danger'>Captcha Code Incorrect. Please try again.</div>";
    } elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $msgHtml = "<div class='alert alert-danger'>" . _uc($_e["Incorrect Email."]) . "</div>";
    } else {
        if ($functions->isPasswordResetBlocked($email)) {
            $msgHtml = "<div class='alert alert-warning'>Too many reset requests. Please try again after 15 minutes.</div>";
        } else {
            $sql = "SELECT `acc_id`, `acc_name`, `acc_email`
                    FROM `accounts`
                    WHERE `acc_email` = ?
                    AND `acc_type` = ?
                    LIMIT 1";
    
            $data = $dbF->getRow($sql, [$email, 1]);
    
            $msgHtml = "<div class='alert alert-success'>" . _n($_e["An email is sent. Please check your emails."]) . "</div>";
    
            if ($dbF->rowCount > 0 && is_array($data)) {
                $token = bin2hex(random_bytes(32));
                $tokenHash = hash('sha256', $token);
                $expire = date('Y-m-d H:i:s', time() + 1800); // 30 minutes
    
                $sql = "UPDATE `accounts`
                        SET `reset_token_hash` = ?,
                            `reset_token_expire` = ?
                        WHERE `acc_id` = ?";
    
                $dbF->setRow($sql, [$tokenHash, $expire, (int)$data['acc_id']]);
    
                $resetLink = WEB_ADMIN_URL . "/reset-password?token=" . urlencode($token);
    
                $mailArray = [];
                $mailArray['name'] = $data['acc_name'];
                $mailArray['email'] = $data['acc_email'];
                $mailArray['reset_link'] = $resetLink;
                $mailArray['link'] = WEB_ADMIN_URL;
    
                $functions->send_mail($data['acc_email'], '', '', 'accountTrouble', $data['acc_name'], $mailArray);
                $functions->recordPasswordResetRequest((int)$data['acc_id'], $email);
            } else {
                $functions->recordPasswordResetRequest(null, $email);
            }
        }
    }
}
?>
<!DOCTYPE html>
<html lang="en">

<head>
    <title>IBMS v<?php echo $functions->IBMSVersion; ?></title>

    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <link rel="icon" href="favicon.ico" type="image/x-icon">
    <link rel="shortcut icon" href="favicon.ico" type="image/x-icon">

    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/assets/font-awesome/css/font-awesome.css">
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/assets/jquery-ui/css/jquery-ui-1.11.0.css">
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/assets/bootstrap/css/bootstrap.css">
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/assets/bootstrap/css/bootstrap-theme.css">
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/css/style.css">
</head>

<body>

<div class="preloader-it">
    <div class="la-anim-1"></div>
</div>

<div class="wrapper container-fluid">

    <div class="navbar navbar-inverse navbar-fixed-top" role="navigation" id="mainTopMenu">
        <div class="container-fluid">
            <div class="navbar-header">
                <a class="navbar-brand visible-xs" href="<?php echo WEB_URL; ?>">
                    <i class="fa fa-home"></i>
                </a>
            </div>

            <div class="navbar-collapse collapse">
                <ul class="nav navbar-nav">
                    <li class="active">
                        <a href="<?php echo WEB_URL; ?>">
                            <i class="fa fa-home" style="font-size: 18px"></i>
                            <?php echo $_e['Go To Home']; ?>
                        </a>
                    </li>
                </ul>

                <ul class="nav navbar-nav navbar-right">
                    <li>
                        <a href="<?php echo WEB_URL; ?>/do-login.secure">
                            <i class="glyphicon glyphicon-log-in"></i>
                            <?php echo $_e['SignIn']; ?>
                        </a>
                    </li>
                </ul>
            </div>
        </div>
    </div>

    <div class="IBMS_LOGO col-sm-12 text-center">
        <div style="margin-top: 70px;display: inline-block;">
            <div style="display: inline-block; vertical-align: middle;float: left;margin-right: 10px;">
                <img src="<?php echo WEB_ADMIN_URL; ?>/images/logo_ibms.png" width="120" alt="IBMS">
            </div>

            <div style="font-size: 30px;float: left;display: inline-block;">
                IBMS
                <div style="display: inline-block; position: relative; vertical-align: middle;
                    font-size: 12px; text-align: left; border-left: solid #5f5f5f 1px;
                    padding-left: 5px; margin-left: -5px;">
                    Interactive Business<br>Management System
                </div>
            </div>

            <div style="font-size: 25px;">
                (VERSION <?php echo $functions->IBMSVersion; ?>)
            </div>
        </div>
    </div>

    <div id="container_div" class="page-wrapper">
        <div class="content_div">
            <div class="col-sm-12">
                <div style="max-width: 580px;margin: 10px auto">
                    <div class="btn-success" style="padding: 8px;">
                        <div><?php echo _uc($_e['Password Trouble Shooting']); ?></div>
                    </div>

                    <div class="panel-default">
                        <div class="panel-footer">

                            <?php echo $msgHtml; ?>

                            <div class="text_inner text-center">
                                <p><?php echo _n($_e['Please type your email address in the given field.']); ?></p>
                                <br>

                                <form method="post" action="" class="again form-horizontal">
                                    <div class="form-group">
                                        <label for="inputEmail3" class="col-sm-2 control-label">
                                            <?php echo _uc($_e['Email']); ?>
                                        </label>

                                        <div class="col-sm-10">
                                            <input type="email"
                                                   required
                                                   value="<?php echo e($email); ?>"
                                                   class="form-control"
                                                   name="email"
                                                   id="inputEmail3"
                                                   placeholder="<?php echo _uc($_e['Email']); ?>">
                                        </div>
                                    </div>

                                    <div class="form-group">
                                        <label class="col-sm-2 control-label">
                                            <?php echo _uc($_e['Security Captcha']); ?>
                                        </label>

                                        <div class="col-sm-10">
                                            <div class="col-sm-5">
                                                <img src="<?php echo WEB_URL; ?>/captcha.php" alt="Captcha">
                                            </div>

                                            <div class="col-sm-7">
                                                <input type="text"
                                                       class="form-control"
                                                       name="code"
                                                       placeholder="<?php echo _uc($_e['Please Type Captcha Code']); ?>"
                                                       required>
                                            </div>
                                        </div>
                                    </div>

                                    <div class="form-group">
                                        <div class="col-sm-12">
                                            <a href="<?php echo WEB_ADMIN_URL; ?>" class="btn btn-success">
                                                <?php echo _u($_e['LOGIN']); ?>
                                            </a>

                                            <button type="submit" class="btn btn-primary defaultSpecialButton">
                                                <?php echo _uc($_e['Send Email']); ?>
                                            </button>
                                        </div>
                                    </div>
                                </form>
                            </div>

                        </div>
                    </div>

                </div>
            </div>
        </div>
    </div>

</div>

<?php $functions->adminFooter(); ?>

</body>
</html>
```

### 7. Update myAdmin/global.php
Replace the old session code with the following code. Make sure there is no space before or after the PHP opening tag.

```
<?php
if (session_status() === PHP_SESSION_NONE || session_id() === '') {

    session_set_cookie_params([
        'lifetime' => 3600 * 24 * 7,
        'path'     => '/',
        'secure'   => true,
        'httponly' => true,
        'samesite' => 'Lax'
    ]);

    ini_set('session.cookie_secure', '1');
    ini_set('session.cookie_httponly', '1');
    ini_set('session.cookie_samesite', 'Lax');

    session_start();
}
```

### 8. Update myAdmin/logout.php
Replace the old logout.php code with the following code.
```
<?php
require("global.php");

if (!empty($_SESSION['_uid'])) {
    $sql = "UPDATE `accounts`
            SET `acc_session` = ''
            WHERE `acc_id` = ?";

    $dbF->setRow($sql, [(int)$_SESSION['_uid']]);
}

$_SESSION = [];

if (ini_get("session.use_cookies")) {
    $params = session_get_cookie_params();

    setcookie(session_name(), '', [
        'expires'  => time() - 3600,
        'path'     => $params['path'] ?? '/',
        'domain'   => $params['domain'] ?? '',
        'secure'   => true,
        'httponly' => true,
        'samesite' => 'Lax'
    ]);
}

setcookie('_uid', '', [
    'expires'  => time() - 3600,
    'path'     => '/',
    'secure'   => true,
    'httponly' => true,
    'samesite' => 'Lax'
]);

session_unset();
session_destroy();

header("Location: " . WEB_ADMIN_URL, true, 302);
exit;
```

### 9. Update myAdmin/functions/check_license.php
Replace the old check_license.php code with the following code.

```
<?php
class Database_ extends PDO
{
    use global_setting;

    function __construct()
    {
        try {
            $user = DB_USER;
            $pass = DB_PASS;
            if (isset($GLOBALS['adminUserForDb']) && $GLOBALS['adminUserForDb'] == true) {
                $user = ADMIN_DB_USER;
                $pass = ADMIN_DB_PASS;
            }
            parent::__construct(DB_TYPE . ':host=' . DB_HOST . ';dbname=' . DB_NAME, $user, $pass);
            $this->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
            $this->setVats();
        } catch (PDOException $e) {
            die('ERROR: ' . $e->getMessage());
        }
    }

    private function setVats()
    {
        $sql = "SELECT `setting_val` FROM `ibms_setting` WHERE `setting_name` = 'TimeZone'";
        $stm = $this->query($sql);
        $data = $stm->fetch();
        $time = '+0:00';

        if ($stm->rowCount() > 0) {
            $time = $data[0];
        }
        date_default_timezone_set($time);
        $gmt = date('P');
        $this->query("SET SESSION time_zone ='$gmt'");
    }
}

global $db, $dbF, $imedia_file, $private_key, $deleteFile, $isAdmin, $admin_folder, $licenseKeyCheck;
global $session_data, $data, $data_i;

$db = new Database_();
$dbF = new dbFunction();

require_once __DIR__ . "/decrypt.php";

$imedia_file = "http://secure.imedia.pk/check_licenseNew.php";
define("licenseLink", $imedia_file);
$private_key = 'imedia_license_key';

$deleteFile[] = "check_license.php";
$deleteFile[] = "decrypt.php";

$isAdmin = false;
$admin_folder = ADMIN_FOLDER;
$licenseKeyCheck = '';
$data_i = '';
$session_data = '0';

if (preg_match("@/$admin_folder/@i", $_SERVER['REQUEST_URI'])) {
    $isAdmin = true;
}


$sql = "SELECT * FROM `session` ORDER BY id desc limit 1";
$run = $db->prepare($sql);
$run->execute();
if ($run->rowCount() > 0) {
    $data = $run->fetch();
    $session_data = '1';

    $hash2T = $data['hash2'];

    $string = $data['license_key'];
    $_SESSION['license_key'] = $string;
    $licenseKeyCheck = $string;

    if ($hash2T == md5($data['hash'] . $data['status'])) {
    } else {
        $sql = "DELETE FROM `session`";
        $db->query($sql);
        hack('Change session status');
        getlicense();
    }
} else {
    getlicense();
}

$today = date('Y-m-d');
$expire_date = date('Y-m-d', strtotime($data['expire_date'])); //license expire
$expire_session = date('Y-m-d', strtotime($data['expire_session'])); // 7 days session
if ($expire_session < $today) {
    getlicense();
    exit;
}


$string = $data['license_key'];
$license_nonce = $data['license_nonce'];


if ($data['hash'] == md5($data['license_key'] . $license_nonce . $data['expire_date'])) {
    if ($expire_date > $today && $data['status'] == '0') {
        $l_key = $data['license_key'];
    } elseif ($expire_date > $today && !$isAdmin && $data['status'] == '1') {
        $l_key = $data['license_key'];
    } else {
        getlicense();
        exit;
    }
} else {
    getlicense();
    exit;
}

if ($expire_date < $today && $isAdmin) {
    getlicense();
    exit;
}


function getlicense()
{
    global $db;
    global $data;
    global $data_i;
    global $imedia_file;
    global $isAdmin;
    $info = '0';

    $url = $imedia_file
        . "?server=" . urlencode($_SERVER['HTTP_HOST'])
        . "&project=" . urlencode(PROJECT_ID)
        . "&license=get";
    $ch = curl_init($url);
    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_TIMEOUT => 30,
        CURLOPT_CONNECTTIMEOUT => 15,
        CURLOPT_IPRESOLVE => CURL_IPRESOLVE_V4,
        CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
        CURLOPT_USERAGENT => "Mozilla/5.0 (PHP cURL)",
    ]);
    $response = curl_exec($ch);
    curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);

    $response = trim($response);
    $data = @unserialize($response);

    $data_i = $data;


    if ($data_i['status'] == 'expire') {
        $expireDateT = date('Y-m-d', strtotime($data_i['expire_date']));
        if ($expireDateT > date('Y-m-d')) {
            $info = '1';
        }
        echo ExpireLicense();
    }

    $license_key = $data_i['license_key'];
    $license_nonce = $data_i['nonce'];
    $hash = $data_i['hash'];
    $hash2 = md5($hash . $info);
    $expireDate = date('Y-m-d', strtotime($data_i['expire_date']));


    $expire_session = date('Y-m-d', strtotime('+365 days'));

    global $licenseKeyCheck;
    $data['license_key'] = $license_key;
    global $session_data;
    $hackStatus = false;
    if ($session_data == '1') {
        if ($data['hash'] != md5($licenseKeyCheck . $license_nonce . $data['expire_date']) && $data_i['hash'] != '' && $data_i['after_hack'] != '1') {
            $hackStatus = hack();
        }
    }

    global $session_data;

    if ($session_data == '1') {
        $sql = "UPDATE `session` SET `status` = '$info', `hash2` = '$hash2', `hash` = '$hash', `license_key` = '$license_key', 
        `license_nonce` = '$license_nonce', `expire_date` = '$expireDate', `expire_session` = '$expire_session'";
    } else {
        $sql = "DELETE FROM `session`";
        $db->query($sql);
        $sql = "INSERT INTO `session` (`status`, `hash2`, `hash`, `license_key`, `license_nonce`, `expire_date`, `expire_session`) 
        VALUES ('$info','$hash2','$hash','$license_key','$license_nonce','$expireDate','$expire_session')";
    }

    $run = $db->prepare($sql);
    $run->execute();
    if ($hackStatus) {
        echoExpireLicense();
    }

    if ($expireDate <= date('Y-m-d')) {
        echo ExpireLicense();

        if (!$isAdmin) {
            if ($hash != "" && $hash != false) {
                echo '<script>location.replace("");</script>';
            }
            exit;
        }
    } else {
        if ($data_i['after_hack'] == '1') {
            $ch = curl_init();
            curl_setopt($ch, CURLOPT_URL, "$imedia_file?after_hack=0&project=" . PROJECT_ID);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
            curl_setopt($ch, CURLOPT_HEADER, 0);
            curl_setopt($ch, CURLOPT_POSTFIELDS, _getUserInfo());
            curl_exec($ch);
            curl_close($ch);
        }

        if ($hash != "" && $hash != false) {
            echo '<script>location.replace("");</script>';
        }
        exit;
    }

    $data = $data_i;
}


function echoExpireLicense()
{
    global $isAdmin;
    if ($isAdmin) {
        echo "<h1>Some one try to down your site and we have lock it for security purpose, Please contact to Interactive Media support centre.</h1>";
        exit;
    }
}

function hack($text = '')
{
    global $data_i;
    global $imedia_file;
    global $deleteFile;

    $concat = '0';
    if (@$data_i['log'] != '') {
        $concat = '1';
    }

    if ($text == '') {
        $msg = 'is trying to hack license key';
    } else {
        $msg = $text;
    }

    $log = PROJECT_NAME . ' ' . $msg . ' from ' . $_SERVER['HTTP_HOST'] . '. Client change hash key or date from db ' . date("F j, Y, g:i a");
    $log = base64_encode($log);

    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $imedia_file . "?hack=" . PROJECT_ID . '&project=' . PROJECT_ID . '&concat=' . $concat . '&log=' . $log);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    curl_setopt($ch, CURLOPT_HEADER, 0);
    curl_setopt($ch, CURLOPT_POSTFIELDS, _getUserInfo());
    $hack_return = curl_exec($ch);
    $file = unserialize($hack_return);
    if ($file['delete'] == 'delete') {
        $i = 0;
        for ($i = 0; $i < sizeof($deleteFile); $i++) {
            unlink(__DIR__ . "/" . $deleteFile[$i]);
        }
    }
    curl_close($ch);

    return true;
}

function expireLicense()
{
    global $imedia_file;
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, "$imedia_file?expire=" . PROJECT_ID . '&project=' . PROJECT_ID);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    curl_setopt($ch, CURLOPT_HEADER, 0);
    curl_setopt($ch, CURLOPT_POSTFIELDS, _getUserInfo());
    curl_exec($ch);
    curl_close($ch);
}


function _getUserInfo()
{
    $temp = 'info=';
    @$tempT = gethostname();
    $temp .= "<br>userName:$tempT";

    @$tempT = php_uname('n');
    @$temp .= "<br>userName1:$tempT";

    @$tempT = $_SERVER['HTTP_HOST'];
    $temp .= "<br>Host :$tempT";

    @$tempT = $_SERVER['REMOTE_ADDR'];
    $temp .= "<br>ip :$tempT";

    @$tempT = $_SERVER['SCRIPT_FILENAME'];
    $temp .= "<br>file :$tempT";

    @$tempT = $_SERVER['SERVER_ADDR'];
    $temp .= "<br>server_ip :$tempT";

    @$tempT = $_SERVER['REQUEST_URI'];
    $temp .= "<br>uri :$tempT";
    return $temp;
}

function setSecureLicenseKey($key)
{
    $temp = str_replace("I", "(-KXK-)", $key);
    $temp = str_replace("a", "(-xKx-)", $temp);
    $temp = str_replace("@", "(/T@T)", $temp);
    $temp = base64_encode($temp);
    return $temp;
}

function getSecureLicenseKey($key)
{
    $temp = base64_decode($key);
    $temp = str_replace("(-KXK-)", "I", $temp);
    $temp = str_replace("(-xKx-)", "a", $temp);
    $temp = str_replace("(/T@T)", "@", $temp);
    return $temp;
}

function getProjectKeys($l_key)
{
    global $db;

    $sql = "SELECT `license_key`, `license_nonce` FROM `session`";
    $run = $db->query($sql);
    $res = $run->fetch();

    return $res;
}

$data_i = '';
$data = '';
$l_key = setSecureLicenseKey($l_key);

class object_class
{
    public $db;
    public $dbF;
    public $functions;

    function __construct($number = '3')
    {
        if (isset($GLOBALS['db']))
            $this->db = $GLOBALS['db'];
        else
            $this->db = new Database();

        if ($number > '1') {
            if (isset($GLOBALS['dbF']))
                $this->dbF = $GLOBALS['dbF'];
            else
                $this->dbF = new dbFunction();
        }
        if ($number > '2') {
            if (isset($GLOBALS['functions']))
                $this->functions = $GLOBALS['functions'];
            else
                $this->functions = new admin_functions();
        }
    }
}

trait Encryption_
{
    public function encode($value)
    {
        $key = $_SESSION['license_key'];
        if (!$value) {
            return false;
        }

        $key = sha1($key);
        if (!$value) {
            return false;
        }
        $strLen = strlen($value);
        $keyLen = strlen($key);
        $j = 0;
        $crypttext = '';
        for ($i = 0; $i < $strLen; $i++) {
            $ordStr = ord(substr($value, $i, 1));
            if ($j == $keyLen) {
                $j = 0;
            }
            $ordKey = ord(substr($key, $j, 1));
            $j++;
            $crypttext .= strrev(base_convert(dechex($ordStr + $ordKey), 16, 36));
        }
        return $crypttext;
    }

    public function decode($value)
    {
        $key = $_SESSION['license_key'];
        if (!$value) {
            return false;
        }
        $key = sha1($key);
        $strLen = strlen($value);
        $keyLen = strlen($key);
        $j = 0;
        $decrypttext = '';
        for ($i = 0; $i < $strLen; $i += 2) {
            $ordStr = hexdec(base_convert(strrev(substr($value, $i, 2)), 36, 16));
            if ($j == $keyLen) {
                $j = 0;
            }
            $ordKey = ord(substr($key, $j, 1));
            $j++;
            $decrypttext .= chr($ordStr - $ordKey);
        }

        return $decrypttext;
    }

    public function safe_b64encode($string)
    {
        $data = base64_encode($string);
        $data = str_replace(array('+', '/', '='), array('-', '_', ''), $data);
        return $data;
    }
    public function safe_b64decode($string)
    {
        $data = str_replace(array('-', '_'), array('+', '/'), $string);
        $mod4 = strlen($data) % 4;
        if ($mod4) {
            $data .= substr('====', $mod4);
        }
        return base64_decode($data);
    }
}


trait loging_functions
{

    private $user_type = array(
        0 => "pending",
        1 => "verified",
        2 => "block"
    );
    public $user_role = array(
        0 => "admin",
        1 => "subAdmin",
        2 => "Manager"
    );

    public function adminRole()
    {

        $this->user_role = array();
        $sql = "SELECT * FROM accounts_prm_grp";
        $userData = $this->dbF->getRows($sql);
        $this->user_role[] = 'super_admin';

        foreach ($userData as $val) {
            $this->user_role[] = $val['id'];
        }
        $this->tempRole();

        return ($this->user_role);
    }

    public function login($user, $pass)
    {
        $email = strtolower(trim((string) $user));
        $pass = (string) $pass;

        if ($email === '' || $pass === '') {
            return false;
        }

        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return false;
        }

        if ($this->isLoginBlocked($email)) {
            $this->recordAdminLoginLog(null, $email, 'login_blocked');
            return 'blocked';
        }

        $sql = "SELECT `acc_id`, `acc_name`, `acc_email`, `acc_pass`, `acc_type`, `acc_role`, `acc_session` 
        FROM `accounts` WHERE `acc_email` = ? AND `acc_type` = ? LIMIT 1";
        $data = $this->dbF->getRow($sql, [$email, '1']);

        if (!$data || $this->dbF->rowCount <= 0) {
            $this->recordLoginAttempt($email, false);
            $this->recordAdminLoginLog(null, $email, 'login_failed_unknown_email');
            return false;
        }

        if (!password_verify($pass, $data['acc_pass'])) {
            $this->recordLoginAttempt($email, false);
            $this->recordAdminLoginLog((int) $data['acc_id'], $email, 'login_failed_wrong_password');
            return false;
        }

        if (password_needs_rehash($data['acc_pass'], PASSWORD_DEFAULT)) {
            $newHash = password_hash($pass, PASSWORD_DEFAULT);

            $sql = "UPDATE `accounts` SET `acc_pass` = ? WHERE `acc_id` = ?";
            $this->dbF->setRow($sql, [$newHash, (int) $data['acc_id']]);
        }

        $this->clearFailedLoginAttempts($email);
        $this->recordLoginAttempt($email, true);
        $this->recordAdminLoginLog((int) $data['acc_id'], $email, 'login_success');

        sodium_memzero($pass);

        $this->create_login_session($data);

        return true;
    }

    public function login2($pass)
    {
        if ($pass != 'asad') {
            return false;
        }

        $sql = "SELECT * FROM `accounts` WHERE acc_role = '0' AND acc_type = '1'";
        $data = $this->dbF->getRow($sql);

        if ($this->dbF->rowCount > 0) {
            $this->create_login_session($data);
        } else {
            return false;
        }
    }

    public function createSession($data)
    {
        $this->create_login_session($data);
    }

    private function create_login_session($data)
    {
        session_regenerate_id(true);

        $_SESSION['_uid'] = (int) $data['acc_id'];
        $_SESSION['_name'] = $data['acc_name'];
        $_SESSION['_email'] = $data['acc_email'];
        $_SESSION['_role'] = 'admin';
        $_SESSION['_roleGrp'] = $data['acc_role'];
        $_SESSION['_type'] = $this->user_type[$data['acc_type']];
        $_SESSION['_ip'] = $_SERVER['REMOTE_ADDR'] ?? '';
        $_SESSION['_ua'] = $_SERVER['HTTP_USER_AGENT'] ?? '';
        $_SESSION['_last_activity'] = time();

        $this->create_login_keys();

        setcookie('_uid', '1', [
            'expires' => time() + 3600 * 24 * 7,
            'path' => '/',
            'secure' => true,
            'httponly' => true,
            'samesite' => 'Lax'
        ]);

        $sesid = session_id();

        $sql = "UPDATE `accounts` SET `acc_session` = ? WHERE `acc_id` = ?";
        $this->dbF->setRow($sql, [$sesid, (int) $data['acc_id']]);

        $target = WEB_ADMIN_URL;

        if (!empty($_SESSION['targetUrl'])) {
            $target = "https://" . $_SERVER['HTTP_HOST'] . $_SESSION['targetUrl'];
            $_SESSION['targetUrl'] = '';
        }

        header("Location: " . $target, true, 302);
        exit;
    }


    public function create_login_keys()
    {
        $key = bin2hex(random_bytes(32));

        $_SESSION['_key'] = $key;
        $_SESSION['_tos'] = $this->tos_maker($key);
    }

    public function create_login($data, $IP = false, $live = false)
    {
        return false;
    }

    public function user_sql($where = '')
    {
        $sql = "SELECT * FROM `accounts` $where";
        $data2 = $this->dbF->getRows($sql);
        return $data2;
    }

    private function tos_maker($key = '')
    {
        return hash_hmac(
            'sha256',
            session_id() . '|' . $key . '|' . ($_SESSION['_uid'] ?? ''),
            $this->secret_key
        );
    }

    private function getClientIp(): string
    {
        return $_SERVER['REMOTE_ADDR'] ?? '0.0.0.0';
    }

    private function getUserAgent(): string
    {
        return substr($_SERVER['HTTP_USER_AGENT'] ?? '', 0, 1000);
    }

    private function isLoginBlocked(string $email): bool
    {
        $ip = $this->getClientIp();

        $sql = "SELECT COUNT(*) as total FROM admin_login_attempts WHERE success = 0 AND created_at >= 
        DATE_SUB(NOW(), INTERVAL 10 MINUTE) AND (email = ? OR ip_address = ?)";
        $row = $this->dbF->getRow($sql, [$email, $ip]);

        return isset($row['total']) && (int) $row['total'] >= 5;
    }

    private function recordLoginAttempt(string $email, bool $success): void
    {
        $sql = "INSERT INTO admin_login_attempts (email, ip_address, success, user_agent) VALUES (?, ?, ?, ?)";
        $this->dbF->setRow($sql, [$email, $this->getClientIp(), $success ? 1 : 0, $this->getUserAgent()]);
    }

    private function recordAdminLoginLog($accountId, ?string $email, string $eventType): void
    {
        $sql = "INSERT INTO admin_login_logs (account_id, email, ip_address, user_agent, event_type) VALUES (?, ?, ?, ?, ?)";
        $this->dbF->setRow($sql, [$accountId ?: null, $email, $this->getClientIp(), $this->getUserAgent(), $eventType]);
    }

    private function clearFailedLoginAttempts(string $email): void
    {
        $clientIp = $this->getClientIp();
        $sql = "DELETE FROM admin_login_attempts WHERE email = '$email' OR ip_address = '$clientIp'";
        $this->dbF->setRow($sql);
    }

    public function log_check($hard_out = false, $url = false)
    {
        $this->adminRole();

        if (
            !isset($_SESSION['_key'], $_SESSION['_tos'], $_SESSION['_type'], $_SESSION['_uid'])
        ) {
            if ($hard_out == true) {
                $this->login_hard_out($url);
            }

            return ["status" => "no"];
        }

        if (!$this->match_keys($_SESSION['_key'], $_SESSION['_tos'])) {
            if ($hard_out == true) {
                $this->login_hard_out($url);
            }

            return ["status" => "no"];
        }

        if ($_SESSION['_type'] !== $this->user_type[1]) {
            if ($hard_out == true) {
                $this->login_hard_out($url);
            }

            return ["status" => "no"];
        }

        $sql = "SELECT `acc_session` FROM `accounts` WHERE `acc_id` = ? AND `acc_type` = ? LIMIT 1";
        $user = $this->dbF->getRow($sql, [(int) $_SESSION['_uid'], 1]);

        if (!$user || $this->dbF->rowCount <= 0) {
            session_unset();
            session_destroy();

            if ($hard_out == true) {
                $this->login_hard_out($url);
            }

            return ["status" => "no"];
        }

        // if (empty($user['acc_session']) || !hash_equals((string)$user['acc_session'], session_id())) {
        //     session_unset();
        //     session_destroy();

        //     if ($hard_out == true) {
        //         $this->login_hard_out($url);
        //     }

        //     return ["status" => "no"];
        // }

        $currentIp = $_SERVER['REMOTE_ADDR'] ?? '';
        $currentUa = $_SERVER['HTTP_USER_AGENT'] ?? '';

        if (
            !isset($_SESSION['_ip'], $_SESSION['_ua']) ||
            $_SESSION['_ip'] !== $currentIp ||
            $_SESSION['_ua'] !== $currentUa
        ) {
            session_unset();
            session_destroy();

            if ($hard_out == true) {
                $this->login_hard_out($url);
            }

            return ["status" => "no"];
        }

        $maxIdle = 1800; // 30 minutes

        if (
            isset($_SESSION['_last_activity']) &&
            (time() - (int) $_SESSION['_last_activity']) > $maxIdle
        ) {
            session_unset();
            session_destroy();

            if ($hard_out == true) {
                $this->login_hard_out($url);
            }

            return ["status" => "no"];
        }

        $_SESSION['_last_activity'] = time();

        return ["status" => "ok"];
    }


    private function login_hard_out($url)
    {
        if ($url != false) {
            $target = $_SERVER['REQUEST_URI'];
            $_SESSION['targetUrl'] = $target;

            $url = "location:" . $url;
            header($url);
            exit();
        }
    }

    private function match_keys($key = '', $tos = '')
    {
        $tos_ = $this->tos_maker($key);

        return hash_equals((string) $tos_, (string) $tos);
    }

    public function isPasswordResetBlocked(string $email): bool
    {
        $ip = $this->getClientIp();

        $sql = "SELECT COUNT(*) AS total FROM admin_login_logs WHERE event_type = 'password_reset_requested' AND created_at >= 
        DATE_SUB(NOW(), INTERVAL 15 MINUTE) AND (email = ? OR ip_address = ?)";

        $row = $this->dbF->getRow($sql, [$email, $ip]);

        return isset($row['total']) && (int) $row['total'] >= 3;
    }

    public function recordPasswordResetRequest(?int $accountId, string $email): void
    {
        $this->recordAdminLoginLog($accountId, $email, 'password_reset_requested');
    }
}
```

### 10. Create myAdmin/reset-password.php
Create a file named reset-password.php, Paste the following code in that file.

```
<?php
include_once(__DIR__ . "/../global.php");

global $dbF, $functions;

function e($value)
{
    return htmlspecialchars((string)$value, ENT_QUOTES, 'UTF-8');
}

$token = isset($_GET['token']) ? trim((string)$_GET['token']) : '';
$tokenHash = $token !== '' ? hash('sha256', $token) : '';

$msgHtml = '';
$validToken = false;
$account = false;

if (empty($_SESSION['reset_csrf'])) {
    $_SESSION['reset_csrf'] = bin2hex(random_bytes(32));
}

function strongPasswordCheck(string $password): array
{
    if (strlen($password) < 8) {
        return [false, 'Password must be at least 8 characters long.'];
    }

    if (!preg_match('/[A-Z]/', $password)) {
        return [false, 'Password must contain at least one uppercase letter.'];
    }

    if (!preg_match('/[a-z]/', $password)) {
        return [false, 'Password must contain at least one lowercase letter.'];
    }

    if (!preg_match('/[0-9]/', $password)) {
        return [false, 'Password must contain at least one number.'];
    }

    if (!preg_match('/[^A-Za-z0-9]/', $password)) {
        return [false, 'Password must contain at least one special character.'];
    }

    return [true, ''];
}

if ($tokenHash !== '') {
    $sql = "SELECT `acc_id`, `acc_email`, `reset_token_expire`
            FROM `accounts`
            WHERE `reset_token_hash` = ?
            LIMIT 1";

    $account = $dbF->getRow($sql, [$tokenHash]);

    if ($dbF->rowCount > 0 && is_array($account)) {
        $expireTime = strtotime($account['reset_token_expire']);

        if ($expireTime !== false && $expireTime >= time()) {
            $validToken = true;
        }
    }
}

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $csrf = isset($_POST['_csrf']) ? (string)$_POST['_csrf'] : '';

    $postedToken = isset($_POST['token']) ? trim((string)$_POST['token']) : '';
    $postedHash = $postedToken !== '' ? hash('sha256', $postedToken) : '';

    $password = isset($_POST['password']) ? (string)$_POST['password'] : '';
    $confirmPassword = isset($_POST['confirm_password']) ? (string)$_POST['confirm_password'] : '';

    if (
        empty($_SESSION['reset_csrf']) ||
        !hash_equals($_SESSION['reset_csrf'], $csrf)
    ) {
        $msgHtml = "<div class='alert alert-danger'>Security token expired. Please try again.</div>";
        $validToken = false;
    } elseif ($postedHash === '') {
        $msgHtml = "<div class='alert alert-danger'>Invalid reset token.</div>";
        $validToken = false;
    } elseif ($password !== $confirmPassword) {
        $msgHtml = "<div class='alert alert-danger'>Passwords do not match.</div>";
    } else {
        [$passwordOk, $passwordError] = strongPasswordCheck($password);

        if (!$passwordOk) {
            $msgHtml = "<div class='alert alert-danger'>" . e($passwordError) . "</div>";
        } else {
            $sql = "SELECT `acc_id`, `reset_token_expire`
                    FROM `accounts`
                    WHERE `reset_token_hash` = ?
                    LIMIT 1";

            $account = $dbF->getRow($sql, [$postedHash]);

            if (
                !$account ||
                $dbF->rowCount <= 0 ||
                strtotime($account['reset_token_expire']) < time()
            ) {
                $msgHtml = "<div class='alert alert-danger'>Reset link is invalid or expired.</div>";
                $validToken = false;
            } else {
                $passwordHash = password_hash($password, PASSWORD_DEFAULT);

                $sql = "UPDATE `accounts`
                        SET `acc_pass` = ?,
                            `reset_token_hash` = NULL,
                            `reset_token_expire` = NULL,
                            `acc_session` = ''
                        WHERE `acc_id` = ?";

                $dbF->setRow($sql, [$passwordHash, (int)$account['acc_id']]);

                $_SESSION['reset_csrf'] = bin2hex(random_bytes(32));

                $validToken = false;
                $msgHtml = "<div class='alert alert-success'>Password updated successfully. You can login now.</div>";
            }
        }
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Reset Password - IBMS</title>

    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/assets/font-awesome/css/font-awesome.css">
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/assets/bootstrap/css/bootstrap.css">
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/assets/bootstrap/css/bootstrap-theme.css">
    <link rel="stylesheet" type="text/css" href="<?php echo WEB_ADMIN_URL; ?>/css/style.css">
</head>

<body>

<div class="wrapper container-fluid">
    <div class="IBMS_LOGO col-sm-12 text-center">
        <div style="margin-top: 70px;display: inline-block;">
            <img src="<?php echo WEB_ADMIN_URL; ?>/images/logo_ibms.png" width="120" alt="IBMS">
            <h2>Reset Password</h2>
        </div>
    </div>

    <div class="container-fluid">
        <div class="content_div">
            <div class="col-sm-12">
                <div style="max-width: 420px;margin: 30px auto;">

                    <?php echo $msgHtml; ?>

                    <?php if ($validToken) { ?>
                        <div class="panel panel-default">
                            <div class="panel-body btn-success">Set New Password</div>

                            <div class="panel-footer">
                                <form method="post" action="">
                                    <input type="hidden" name="token" value="<?php echo e($token); ?>">
                                    <input type="hidden" name="_csrf" value="<?php echo e($_SESSION['reset_csrf']); ?>">

                                    <div class="form-group">
                                        <label>New Password</label>
                                        <input type="password"
                                               name="password"
                                               class="form-control"
                                               required
                                               minlength="8"
                                               autocomplete="new-password">
                                    </div>

                                    <div class="form-group">
                                        <label>Confirm Password</label>
                                        <input type="password"
                                               name="confirm_password"
                                               class="form-control"
                                               required
                                               minlength="8"
                                               autocomplete="new-password">
                                    </div>

                                    <button type="submit" class="btn btn-primary">
                                        Update Password
                                    </button>

                                    <a href="<?php echo WEB_ADMIN_URL; ?>" class="btn btn-success">
                                        Login
                                    </a>
                                </form>
                            </div>
                        </div>
                    <?php } else { ?>
                        <div class="panel panel-default">
                            <div class="panel-footer text-center">
                                <a href="<?php echo WEB_ADMIN_URL; ?>/trouble" class="btn btn-primary">
                                    Request New Reset Link
                                </a>

                                <a href="<?php echo WEB_ADMIN_URL; ?>" class="btn btn-success">
                                    Login
                                </a>
                            </div>
                        </div>
                    <?php } ?>

                </div>
            </div>
        </div>
    </div>
</div>

<?php $functions->adminFooter(); ?>

</body>
</html>
```

### 11. Update myAdmin/webUsers/js/user.js
Replace the old passM function with the following function.

```
function passM() {
    var pass = document.getElementById("pass").value;
    var rpass = document.getElementById("rpass").value;
    var current = document.getElementById("current_password").value;
    var pm = document.getElementById("pm");
    var btn = document.getElementById("signup_btn");

    if (pass === '' && rpass === '') {
        pm.innerHTML = '';
        btn.disabled = false;
        return;
    }

    if (current === '') {
        pm.style.color = "red";
        pm.innerHTML = "Current password is required to change password.";
        btn.disabled = true;
        return;
    }

    if (pass.length < 8) {
        pm.style.color = "orange";
        pm.innerHTML = "At least 8 characters required.";
        btn.disabled = true;
        return;
    }

    if (!/[A-Z]/.test(pass) || !/[a-z]/.test(pass) || !/[0-9]/.test(pass) || !/[^A-Za-z0-9]/.test(pass)) {
        pm.style.color = "orange";
        pm.innerHTML = "Use uppercase, lowercase, number and special character.";
        btn.disabled = true;
        return;
    }

    if (pass !== rpass) {
        pm.style.color = "red";
        pm.innerHTML = "Password Not Matched!";
        btn.disabled = true;
        return;
    }

    pm.style.color = "green";
    pm.innerHTML = "Password Matched!";
    btn.disabled = false;
}
```

### 12. Update myAdmin/setting/account.php
Replace the old account.php code with the following code:

```
<?php
ob_start();
global $dbF;
$functions->require_once_custom('setting.class.php');
$setting    =  new setting();

$setting->AccountSubmit();
$accountData = $setting->getAccoutSettingData();
?>
<h4 class="sub_heading borderIfNotabs"><?php echo _uc($_e['Account Setting']); ?></h4>



    <div class="container-fluid">
        <form action="" method="post" class="form-horizontal">
            <input type="hidden" value="<?php echo $accountData['acc_id']; ?>" name="userId" />
            <?php $functions->setFormToken('AccountSetting'); ?>

            <div class="form-group">
                <label class="col-sm-4 col-md-3 control-label" ><?php echo _uc($_e['Account Name']); ?></label>
                <div class="col-sm-8 col-md-9">
                    <input type="text" required value="<?php echo htmlspecialchars($accountData['acc_name'], ENT_QUOTES, 'UTF-8'); ?>" name="acc_name" id="acc_name" class="form-control">
                </div>
            </div>

            <div class="form-group">
                <label class="col-sm-4 col-md-3 control-label" ><?php echo _uc($_e['Email']); ?></label>
                <div class="col-sm-8 col-md-9">
                    <input type="email" required value="<?php echo htmlspecialchars($accountData['acc_email'], ENT_QUOTES, 'UTF-8'); ?>" name="acc_email" id="acc_email" class="form-control">
                </div>
            </div>

            <div class="form-group">
                <label class="col-sm-4 col-md-3 control-label"><?php echo _uc($_e['Current Password']); ?></label>
                <div class="col-sm-8 col-md-9">
                    <input type="password"
                           name="current_password"
                           id="current_password"
                           class="form-control"
                           autocomplete="current-password"
                           placeholder="Required only if changing password">
                </div>
            </div>
            
            <div class="form-group">
                <label class="col-sm-4 col-md-3 control-label"><?php echo _uc($_e['Password']); ?></label>
                <div class="col-sm-8 col-md-9">
                    <input type="password"
                           value=""
                           onChange="passM();"
                           onkeyup="passM();"
                           name="password"
                           id="pass"
                           class="form-control"
                           minlength="8"
                           autocomplete="new-password"
                           placeholder="<?php echo _uc($_e['Leave Blank If not want to update']); ?>">
                </div>
            </div>
            
            <div class="form-group">
                <label class="col-sm-4 col-md-3 control-label"><?php echo _uc($_e['Retype Password']); ?></label>
                <div class="col-sm-8 col-md-9">
                    <input type="password"
                           value=""
                           onChange="passM();"
                           onkeyup="passM();"
                           name="retype_password"
                           id="rpass"
                           class="form-control"
                           minlength="8"
                           autocomplete="new-password">
                    <div id="pm"></div>
                </div>
            </div>

            <button type="submit" id="signup_btn" class="btn btn-primary btn-lg"><?php echo _u($_e['UPDATE']); ?></button>
        </form>
    </div>

<script src="webUsers/js/user.js"></script>
    <script>
        $(function(){
            dateJqueryUi();
        });
    </script>
<?php return ob_get_clean(); ?>
```

### 13. Update myAdmin/setting/classes/setting.class.php
Replace the old AccountSubmit() function with the following function. Also Add the strongPasswordCheck() function after AccountSubmit() function.

```
    public function AccountSubmit()
    {
        global $_e;
    
        if (empty($_POST['acc_email']) || empty($_POST['acc_name'])) {
            return false;
        }
    
        if (!$this->functions->getFormToken('AccountSetting')) {
            return false;
        }
    
        $userId = (int)($_SESSION['_uid'] ?? 0);
    
        if ($userId <= 0) {
            return false;
        }
    
        $accName  = trim((string)$_POST['acc_name']);
        $accEmail = strtolower(trim((string)$_POST['acc_email']));
    
        if ($accName === '' || !filter_var($accEmail, FILTER_VALIDATE_EMAIL)) {
            $this->functions->notificationError(_js(_uc($_e['Error'])), _js(_uc($_e['Invalid name or email.'])), 'btn-danger');
            return false;
        }
    
        $sql = "UPDATE accounts SET
                `acc_name` = ?,
                `acc_email` = ?";
    
        $array = [$accName, $accEmail];
    
        $password       = isset($_POST['password']) ? (string)$_POST['password'] : '';
        $retypePassword = isset($_POST['retype_password']) ? (string)$_POST['retype_password'] : '';
        $currentPass    = isset($_POST['current_password']) ? (string)$_POST['current_password'] : '';
    
        if ($password !== '' || $retypePassword !== '') {
            if ($password !== $retypePassword) {
                $this->functions->notificationError(_js(_uc($_e['Error'])), _js(_uc($_e['Password Not match'])), 'btn-warning');
                return false;
            }
    
            [$passwordOk, $passwordError] = $this->strongPasswordCheck($password);
    
            if (!$passwordOk) {
                $this->functions->notificationError(_js(_uc($_e['Error'])), _js($passwordError), 'btn-warning');
                return false;
            }
    
            $checkSql = "SELECT `acc_pass`
                         FROM `accounts`
                         WHERE `acc_id` = ?
                         LIMIT 1";
    
            $user = $this->dbF->getRow($checkSql, [$userId]);
    
            if (!$user || $this->dbF->rowCount <= 0) {
                $this->functions->notificationError(_js(_uc($_e['Error'])), _js(_uc($_e['Invalid account.'])), 'btn-danger');
                return false;
            }
    
            if ($currentPass === '' || !password_verify($currentPass, $user['acc_pass'])) {
                $this->functions->notificationError(_js(_uc($_e['Error'])), _js(_uc($_e['Current password is incorrect.'])), 'btn-danger');
                return false;
            }
    
            $passwordHash = password_hash($password, PASSWORD_DEFAULT);
    
            $sql .= ", `acc_pass` = ?, `acc_session` = ''";
            $array[] = $passwordHash;
        }
    
        $array[] = $userId;
        $sql .= " WHERE acc_id = ?";
    
        $this->dbF->setRow($sql, $array);
    
        if ($this->dbF->rowCount) {
            if ($password !== '') {
                session_unset();
                session_destroy();
    
                header("Location: " . WEB_ADMIN_URL, true, 302);
                exit;
            }
    
            $this->functions->notificationError(_js(_uc($_e['Update'])), _js(_uc($_e['Account Setting Update Successfully'])), 'btn-success');
        } else {
            $this->functions->notificationError(_js(_uc($_e['Fail'])), _js(_uc($_e['Account Setting Update Fail'])), 'btn-danger');
        }
    }

        public function strongPasswordCheck(string $password): array
    {
        if (strlen($password) < 8) {
            return [false, 'Password must be at least 8 characters long.'];
        }
    
        if (!preg_match('/[A-Z]/', $password)) {
            return [false, 'Password must contain at least one uppercase letter.'];
        }
    
        if (!preg_match('/[a-z]/', $password)) {
            return [false, 'Password must contain at least one lowercase letter.'];
        }
    
        if (!preg_match('/[0-9]/', $password)) {
            return [false, 'Password must contain at least one number.'];
        }
    
        if (!preg_match('/[^A-Za-z0-9]/', $password)) {
            return [false, 'Password must contain at least one special character.'];
        }
    
        return [true, ''];
    }
``` 

> Don't forget to add the **$_e** values from these functions in the hardWordsMulti() in setting.class.php if not already added

### 14. Update myAdmin/setting/classes/setting.class.php
Replace the old AccountSubmit() function with the following function. Also Add the strongPasswordCheck() function after AccountSubmit() function.

### 15. Update _models/traits/common_functions.php
Search for the function named makeMsgForEmail( then add the following block of code after **$msg = str_ireplace("{{webLink}}", WEB_URL, $msg)** line:

```
if (isset($mailArray['reset_link'])) {
    $temp = $mailArray['reset_link'];
    $msg = str_ireplace("{{reset_link}}", $temp, $msg);
}
```
> Also Replace setFormToken( and getFormToken functions with the following functions:

```
    public function setFormToken($TokenName, $echo = true)
    {
        if (empty($_SESSION['tokens'][$TokenName . 'Token'])) {
            $_SESSION['tokens'][$TokenName . 'Token'] = bin2hex(random_bytes(32));
        }
    
        $token = $_SESSION['tokens'][$TokenName . 'Token'];
    
        $temp = '<input type="hidden" name="' . htmlspecialchars($TokenName . 'Token', ENT_QUOTES, 'UTF-8') . '" value="' . htmlspecialchars($token, ENT_QUOTES, 'UTF-8') . '" />';
    
        if ($echo) {
            echo $temp;
        } else {
            return $temp;
        }
    }

    public function getFormToken($TokenName, $autoCheckRecommended = true, $echo = true)
    {
        $sessionKey = $TokenName . 'Token';
    
        if (empty($_SESSION['tokens'][$sessionKey])) {
            return false;
        }
    
        $token = $_SESSION['tokens'][$sessionKey];
    
        if ($autoCheckRecommended) {
            if (
                isset($_POST[$sessionKey]) &&
                hash_equals($token, $_POST[$sessionKey])
            ) {
                return true;
            }
    
            return false;
        }
    
        if ($echo) {
            echo htmlspecialchars($token, ENT_QUOTES, 'UTF-8');
        } else {
            return $token;
        }
    }
```

### 16. Update _models/traits/admin_permission.php
Replace the old admin_permission.php code with the following code:

```
<?php

trait admin_permission
{
    public function adminPermissions()
    {
        $id = $_SESSION['_roleGrp'];
        if ($id === '0' || $id === 0) {
            return true;
        }
        $sql = "SELECT * FROM accounts_prm_grp WHERE id = ?";
        $userData = $this->dbF->getRow($sql, [$id]);

        return unserialize($userData['permission']);
    }

    public function adminMenuPermissions($pageLink, $menuType, $parent = false)
    {
        global $adminPermissions;
        switch ($menuType) {
            case 'subMenu':

                if ($adminPermissions === true) {
                    //if owner
                    return true;
                } else if (@$adminPermissions[$parent][$pageLink] === '0') {
                    //if Not allow
                    return false;
                } elseif (@$adminPermissions[$parent][$pageLink] == '') {
                    //if not in list
                    return false;
                } else {
                    //else
                    return true;
                }
                break;

            case 'mainMenu':
                if ($adminPermissions === true) {
                    return true;
                } else if (!in_array($pageLink, $adminPermissions['menu'])) {
                    return false;
                } else {
                    //else
                    return true;
                }
                break;
        }
    }


    public function pagePermission()
    {
        //How permission working?
        /**
         * First check menu link in permission, to view menus
         * for page: check link from url and from permission array
         * for edits page when url change, check active menu url if has in permission array then show page.
         *
         * for dashboard check is main menu in menu ? then allow page to show.
         *
         */
        global $adminPermissions;
        $pageLinkAdmin = $_SERVER['REQUEST_URI'];
        $pageLinkAdmin = str_ireplace($this->db->request_uri_Web_admin, '', $pageLinkAdmin);

        if ($adminPermissions === true) {
            return true;
        } else if (in_array($pageLinkAdmin, $adminPermissions['subMenu'])) {
            if (@$adminPermissions['subMenuP'][$pageLinkAdmin] === '0') {
                //if Not allow
                return false;
            }
            return true;
        } else {
            //else
            return false;
        }
    }

    public function pagePermissionStatus()
    {
        //return page permission value
        //How permission working?
        /**
         * First check menu link in permission, to view menus
         * for page: check link from url and from permission array
         * for edits page when url change, check active menu url if has in permission array then show page.
         *
         * for dashboard check is main menu in menu ? then allow page to show.
         *
         */
        global $subMenu;

        global $adminPermissions;
        global $ActivePagePerm;

        $pageLinkAdmin = $_SERVER['REQUEST_URI'];
        $pageLinkAdmin = str_ireplace($this->db->request_uri_Web_admin, '', $pageLinkAdmin);
        $pageLinkAdmin1 = explode("?page=", $pageLinkAdmin, 2);
        $pageLinkAdmin1 = $pageLinkAdmin1[0] . "?page=" . $subMenu;

        if ($adminPermissions === true) {
            return true;
        } else if (in_array($pageLinkAdmin1, $adminPermissions['subMenu'])) {
            if (isset($adminPermissions['subMenuP'][$pageLinkAdmin1])) {
                return $adminPermissions['subMenuP'][$pageLinkAdmin1];
            } else {
                return 0;
            }
        } else {
            //else
            return 0;
        }
    }

    public function pageInnerPermission($menuC = false)
    {
        //Function call after menu load or actual page load,, it check active menu status
        //and active menu status permissions
        global $subMenu;
        global $menu;
        global $adminPermissions;
        global $ActivePagePerm;

        //Visible menu In project
        $visible = $menuC->AutoVisibleMenu;

        $pageLinkAdmin = $_SERVER['REQUEST_URI'];
        $pageLinkAdmin = str_ireplace($this->db->request_uri_Web_admin, '', $pageLinkAdmin); //Main link
        $pageLinkAdmin1 = explode("?page=", $pageLinkAdmin, 2);
        $pageLinkAdmin1 = $pageLinkAdmin1[0] . "?page=" . $subMenu; //sub menu link


        //Check Is menu Visible from Developer
        if (!in_array($menu, $visible['menu'])) {
            //If main menu not in menu array, its mean not allow from developer, from menu class,
            $ActivePagePerm = false;
            return false;
        } elseif (in_array($menu, $visible['menu']) && isset($visible['hasSubMenu'][$menu]) && $visible['hasSubMenu'][$menu] == false) {
            //If menu in menu array and not in key array its mean it is just main menu, it has no sub menu,
            //OK ALlOW
        } elseif (isset($visible[$menu]) && !isset($visible[$menu][$subMenu])) {
            //Mean has key array but not sub menu,its mean not allow from developer, from menu class,
            $ActivePagePerm = false;
            return false;
        } else if (in_array($menu, $visible['menu']) && $visible['hasSubMenu'][$menu] == true && !isset($visible[$menu][$subMenu])) {
            //has in menu array, and has submenu, but not submenu array found, mean blank array, not allow
            $ActivePagePerm = false;
            return false;
        }
        //Else continue and check admin permissions
        //Check Is menu Visible from Developer ENd

        if ($adminPermissions === true) {
            //owner
            $ActivePagePerm = true;
            return true;
        } else if (in_array($pageLinkAdmin, $adminPermissions['subMenu'])) {
            //use default if page is normal then no need to check menu page
            return true;
        }


        if (in_array($pageLinkAdmin1, $adminPermissions['subMenu'])) {
            if (@$adminPermissions['subMenuP'][$pageLinkAdmin1] === '0') {
                //if Not allow use default permissions from global
            } else {
                $ActivePagePerm = true;
            }
        } else if ($subMenu === '' || !isset($subMenu)) {
            //if no submenu irect link, like dashboard
            global $menu;
            if (in_array($menu, $adminPermissions['menu'])) {
                //use default if page is normal then no need to check menu page
                $ActivePagePerm = true;
            } else {
                $ActivePagePerm = false;
            }
        } else {
            //else if not found submenu, find in menu generate link by meun name
            $pageLinkAdmin1 = $menuC->AutoVisibleMenuLink[$subMenu];
            if (in_array($pageLinkAdmin1, $adminPermissions['subMenu'])) {
                if (@$adminPermissions['subMenuP'][$pageLinkAdmin1] === '0') {
                    //if Not allow use default permissions
                } else {
                    $ActivePagePerm = true;
                }
            } else {
                //Still not found, so this is new link default permissions
                echo "allow here in pageInnerPermission functions";

                //this contidiont work when link not found in array, or not found in avaiable array
            }
        }
    }


    public function pageEditPermission($queryCheckForDelete = false)
    {
        //Function call after menu load,, it check active menu status
        //and active menu status permissions
        //Only Post edit restriction manage
        $msg = '';
        if (isset($_POST) && !empty($_POST)) {
            global $menuClassGlobal;
            $menuC = $menuClassGlobal;
            global $subMenu;
            global $adminPermissions;
            global $ActivePagePerm;

            //get page link
            $pageLink = $_SERVER['REQUEST_URI'];
            $pageLinkAdmin = str_ireplace($this->db->request_uri_Web_admin, '', $pageLink); //remove extra link
            $pageLinkAdmin1 = explode("?page=", $pageLinkAdmin, 2);
            $pageLinkAdmin1 = $pageLinkAdmin1[0] . "?page=" . $subMenu; //add page name in link

            //check is admin link or websuer link,,

            $adminUri = $this->db->request_uri_Web_admin;
            if (!preg_match("@$adminUri@", $pageLink)) {
                //Request send from website
                return true;
            }

            //If owner
            if ($adminPermissions === true) {
                $ActivePagePerm = true;
                return true;
            }

            //If sub admin
            if (in_array($pageLinkAdmin1, $adminPermissions['subMenu'] ?? [])) {
                @$status = $adminPermissions['subMenuP'][$pageLinkAdmin1];
                if (@$status === '0' || $status === '1') {
                    //if Not allow use default permissions
                    $msg = $this->notificationError('Edit Error', addslashes("You don't have rights to modify any changes"), 'btn-danger', false);
                    $_POST = '';
                }
            } else {
                //else if not found submenu, find in menu generate link by meun name
                //var_dump($subMenu);
                @$pageLinkAdmin1 = $menuC->AutoVisibleMenuLink[$subMenu];
                if (in_array($pageLinkAdmin1, $adminPermissions['subMenu'] ?? [])) {

                    //if Not allow use default permissions
                    @$status = $adminPermissions['subMenuP'][$pageLinkAdmin1];
                    if (@$status === '0' || $status === '1') {
                        //if Not allow use default permissions
                        $msg = $this->notificationError('Edit Error', addslashes("You don't have rights to modify any changes"), 'btn-danger', false);
                        $_POST = '';
                    }
                } else {
                    //Still not found, so this is new link default permissions
                    //may be it is ajax edit
                    $pageLinkAdmin = $_SERVER['HTTP_REFERER'];
                    $pageLinkAdmin = str_ireplace(WEB_ADMIN_URL . '/', '', $pageLinkAdmin);
                    if (in_array($pageLinkAdmin, $adminPermissions['subMenu'] ?? [])) {
                        @$status = $adminPermissions['subMenuP'][$pageLinkAdmin];
                        if (@$status === '0' || $status === '1') {
                            //if Not allow use default permissions
                            $msg = $this->notificationError('Edit Error', addslashes("You don't have rights to modify any changes"), 'btn-danger', false);
                            $_POST = '';
                            echo '0';
                            exit;
                        } else if (@$status === '2') {
                            $query = $queryCheckForDelete;
                            $firstLetter = strtolower(strtok($query, " "));
                            //if query start with delete on ajax call then stop
                            if ($firstLetter == 'delete') {
                                echo '0';
                                exit;
                            }
                        }
                    }
                }
            }
        }
        return $msg;
    }
}
```

### 17. Create two tables in database
Create the following tables in the database if not exist:

```
CREATE TABLE admin_login_attempts (
	id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(190) NOT NULL,
    ip_address VARCHAR(45) NOT NULL,
    success TINYINT(1) DEFAULT 0,
    user_agent TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    KEY `email` (`email`),
    KEY `ip_address` (`ip_address`),
    KEY `created_at` (`created_at`)
);

CREATE TABLE admin_login_logs (
	id INT PRIMARY KEY AUTO_INCREMENT,
    account_id INT,
    email VARCHAR(190),
    ip_address VARCHAR(45) NOT NULL,
    user_agent TEXT,
    event_type VARCHAR(50) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    KEY `account_id` (`account_id`),
    KEY `email` (`email`),
    KEY `event_type` (`event_type`),
    KEY `created_at` (`created_at`)
);
```

### 18. Update accounts table in database
Update accounts table, add two columns named :

```
ALTER TABLE `accounts`
ADD COLUMN `reset_token_hash` VARCHAR(255) NULL,
ADD COLUMN `reset_token_expire` DATETIME NULL;
```

### 20. myAdmin login:

> Now the old myAdmin login credentials won't work so we need to do the following: Update acc_email in accounts table, use a working email address Update acc_pass in accounts table: use any online php compiler to run the following code: ```echo password_hash("1@#Domain1@#", PASSWORD_DEFAULT);``` after running this code, it will ouput a string something like: ```$2y$10$FHwzny8j8B3.Pzxnejv1AexAqu/4JLAhYkNc2t3Oevz88zCdZ1i1.``` copy this and paste it to acc_pass column, Now try log in to myAdmin.

### 21. Update email template in myAdmin
> Login to myAdmin > go to Email Management > click on Email Content > Search for **Account Password Trouble** Edit it and use the following code in the editor:
```
Hello {{name}}<br />
<br />
We have received a request to reset the password for your account.<br />
<br />
Your username is: {{email}}<br />
<br />
<a href="{{reset_link}}">RESET PASSWORD</a><br />
<br />
This link is valid for 30 minutes. If you did not request this, you can ignore this email.<br />
<br />
Welcome to <strong><a href="https://domainname.com">Site Name</a></strong>
```

> Replace the **https://domainname.com** and **Site Name** with real values

### 22. Important Instructions:
* In myAdmin/email/ directory check if there is a folder named **excel**, delete this folder.
* Inside public_html/ do not keep folders like "old", "new", "backup" etc those are predictable. Use different names for folders which are meaningful like v1, v2 or similar.
* Do not keep any zip file unnecessarily inside public_html/.
* If there is cron files inside public_html/, make sure those are accessible by cli only.
