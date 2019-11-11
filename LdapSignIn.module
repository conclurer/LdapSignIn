<?php

class LdapSignIn extends WireData implements Module, ConfigurableModule
{
    protected $ldapSession = null;
    protected $host, $port, $defaultLoginDomain, $userDefaultRoles, $protocol, $debugMode;

    public static function getModuleInfo()
    {
        return array(
            'title' => __('LDAP Sign In'),
            'version' => '051',
            'author' => 'Conclurer GbR',
            'summary' => __('Enables users to sign in via LDAP'),
            'singular' => true,
            'autoload' => true
        );
    }

    public function ___install()
    {
        if (!function_exists('ldap_connect'))
            throw new WireException ('Please make sure that extension php_ldap is loaded.');
    }

    public function init()
    {
        if (isset ($this->data ['host'])) {
            $host = explode(':', str_replace(array(
                'ldap://',
                'ldaps://'
            ), array(
                '',
                ''
            ), $this->data ['host']));
            $this->protocol = (empty ($this->data ['useSSL'])) ? 'ldap://' : 'ldaps://';
            $this->host = $host [0];
            $this->port = empty ($host [1]) ? 389 : $host [1];
        }

        $this->debugMode = empty ($this->data ['debug']);

        if (isset ($this->data ['loginDomain']))
            $this->defaultLoginDomain = $this->data ['loginDomain'];
        $this->userDefaultRoles = new WireArray ();

        if (isset ($this->data ['userDefaultRoles'])) {
            foreach ($this->data ['userDefaultRoles'] as $x) {
                $role = $this->roles->get($x);
                $this->userDefaultRoles->add($role);
            }
        }

        $this->userDefaultRoles->add($this->roles->getGuestRole());
        $this->userDefaultRoles = $this->userDefaultRoles->unique();

        $this->session->addHookAfter('login', $this, 'hookLogin');
        $this->addHook('ProcessLogin::buildLoginForm', $this, 'hookLoginForm');
        $this->addHookAfter('Modules::saveModuleConfigData', $this, 'hookModuleSave');
    }

    public function ___hookLogin(HookEvent $event)
    {
        if ($event->return)
            return; // Skip if already logged in

        $username = $event->arguments [0];
        $password = $event->arguments [1];

        // skip if user already exists locally
        if (!$this->users->get("name=$username") instanceof NullPage)
            return;

        // Set default user domain name if not given
        $username = str_replace('@', '', $username) == $username ? "$username@{$this->defaultLoginDomain}" : $username;

        if ($this->ldapUserLogin($username, $password)) {
            $wireUserName = $this->sanitizer->pageName($username);

            $user = $this->users->get("name=$wireUserName");
            if ($user instanceof NullPage) {
                $usersPath = $this->users->getGuestUser()->parent;

                $user = new User ();
                $user->parent = $usersPath;
                $user->name = $wireUserName;
                $user->pass = $password;
                foreach ($this->userDefaultRoles as $role)
                    $user->addRole($role);
                $user->save();
            } else {
                $user->pass = $password;
                $user->save();
            }

            // Reset given LDAP-Username to WireUsername
            $user = $this->session->login($wireUserName, $password);
            $event->return = $user;

            $this->message($this->_('Logged in via LDAP'));

            return;
        }

        $event->return = null;
        return;
    }

    public function hookLoginForm(HookEvent $event)
    {
        $form = $event->return;
        $defaultDomain = $this->defaultLoginDomain;
        if (empty ($defaultDomain))
            return;

        $bodyField = wire('modules')->get('InputfieldMarkup');
        $bodyField->description = sprintf(__('You will be logged in via domain %1$s if you don\'t supply an different domain name'), "<b>$defaultDomain</b>");
        $form->add($bodyField);

        $event->return = $form;
    }

    public function hookModuleSave(HookEvent $event)
    {
        $className = $event->arguments [0];
        if ($className != get_class($this))
            return;
        $this->validateConfiguration();
    }

    public function ldapUserLogin($user, $password)
    {
        $connection = $this->connectLdap();
        if (!$connection)
            return false;

        $bind = @ldap_bind($connection, $user, $password);

        return ($bind !== false);
    }

    public function validateConfiguration()
    {
        $connection = $this->connectLdap();
        if (!$connection) {
            $this->error($this->_('Unable to connect to LDAP server.'));
            return;
        }

        $this->message($this->_('Successfully connected to LDAP server.'));
    }

    protected function connectLdap()
    {
        if ($this->debugMode)
            ldap_set_option(NULL, LDAP_OPT_DEBUG_LEVEL, 7);
        return ldap_connect($this->protocol . $this->host, $this->port);
    }

    static public function getModuleConfigInputfields(array $data)
    {
        $inputfields = new InputfieldWrapper ();

        $hostField = wire('modules')->get('InputfieldText');
        $hostField->name = 'host';
        $hostField->columnWidth = 80;
        $hostField->label = __('LDAP Server');
        $hostField->required = 1;
        if (isset ($data ['host']))
            $hostField->value = $data ['host'];
        $hostField->description = __('The hostname of your LDAP server. This can be either an ip address or a domain name. Supply a custom port (different than 389) separated with a colon. Examples: 10.0.0.1, controller.domain.com, controller.domain.com:388');
        $inputfields->add($hostField);

        $useSSLField = wire('modules')->get('InputfieldCheckbox');
        $useSSLField->name = 'useSSL';
        $useSSLField->columnWidth = 20;
        $useSSLField->label = __('Use SSL?');
        $useSSLField->description = __('Connects to the LDAP Server via SSL.');
        if (isset ($data ['useSSL']) && $data ['useSSL'] == 1)
            $useSSLField->checked = 1;
        $inputfields->add($useSSLField);

        $defaultLoginDomainField = wire('modules')->get('InputfieldText');
        $defaultLoginDomainField->name = 'loginDomain';
        $defaultLoginDomainField->label = __('Default Login Domain');
        $defaultLoginDomainField->required = 1;
        if (isset ($data ['loginDomain']))
            $defaultLoginDomainField->value = $data ['loginDomain'];
        $defaultLoginDomainField->description = __('This is the domain name used by default if the user does not supply a domain name. It will be added to the username, e.g. username@domainname.com');
        $inputfields->add($defaultLoginDomainField);

        $userDefaultRolesField = wire('modules')->get('InputfieldPageListSelectMultiple');
        $userDefaultRolesField->name = 'userDefaultRoles';
        $userDefaultRolesField->label = __('Default Roles for new Users');
        $userDefaultRolesField->description = __('These user roles will be applied to all new LDAP users. Please note that the guest role is applied automatically.');
        $userDefaultRolesField->parent_id = wire('roles')->getGuestRole()->parent_id;
        if (isset ($data ['userDefaultRoles']))
            $userDefaultRolesField->value = $data ['userDefaultRoles'];
        $inputfields->add($userDefaultRolesField);

        $debugField = wire('modules')->get('InputfieldCheckbox');
        $debugField->name = 'debug';
        $debugField->collapsed = Inputfield::collapsedYes;
        $debugField->label = __('Debug Mode');
        $debugField->description = __('Turns on the debug mode so you can see the output of PHP\'s ldap module in the apache log.');
        if (isset ($data ['debug']) && $data ['debug'] == 1)
            $debugField->checked = 1;
        $inputfields->add($debugField);

        return $inputfields;
    }
}