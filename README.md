### PHP Location Plesk Server
```bash
remote server  > /opt/plesk/php/7.4/bin/php -d memory_limit=-1
```
### User Creation via Shopware CLI
```bash
remote server  > bin/console user:create --admin --email=keanu.klenner@connectiv.de --firstName="keanu" --lastName="klenner" --password=connectiv --no-interaction keanuklenner
```
### Compile Plugin Storefront and Administration Components
#### Compile Storefront Javascript
```bash
docker machine > bin/console plugin:refresh
docker machine > bin/console plugin:install PLUGINNAME
docker machine > bin/console plugin:activate PLUGINNAME

docker machine > copy changed StorefrontJS files to machine
docker machine > ./bin/build-storefront.sh

remote server  > copy compiled minified StorefrontJS to server
location       > plugindir/src/Resources/app/storefront/dist/storefront/js/full-plugin-name.js

remote server  > bin/console theme:compile
remote server  > bin/console cache:clear
```

#### Compile Administration Javascript
```bash
docker machine > bin/console plugin:refresh
docker machine > bin/console plugin:install PLUGINNAME
docker machine > bin/console plugin:activate PLUGINNAME

docker machine > copy changed AdministrationJS files to machine
docker machine > ./bin/build-administration.sh

docker machine > copy compiled minified AdministrationJS to server
location       > plugindir/src/Resources/public/administration/js/full-plugin-name.js

remote server  > bin/console plugin:update PLUGINNAME
remote server  > bin/console theme:compile
remote server  > bin/console cache:clear
```
### Compile Plugin VueJS Components
```bash
docker machine > bin/console plugin:refresh
docker machine > bin/console plugin:install PLUGINNAME
docker machine > bin/console plugin:activate PLUGINNAME

docker machine > copy changed VueJS component files to machine

docker machine > npm run build
location       > plugindir/src/Resources/app/storefront/src/plugins/COMPONENT-NAME

docker machine > ./bin/build-storefront.sh

remote server  > copy compiled minified VueJS to server
location       > plugindir/src/Resources/app/storefront/src/plugins/COMPONENT-NAME/dist/COMPONENT-NAME.js
remote server  > copy compiled minified StorefrontJS to server
location       > plugindir/src/Resources/app/storefront/dist/storefront/js/full-plugin-name.js

remote server  > bin/console theme:compile
remote server  > bin/console cache:clear
```
### How to use EntityRepositories
```php
$criteria = new Criteria();
$criteria->addFilter(new EqualsAnyFilter('[DATA]', [<DATA_ARRAY>]));
$criteria->addAssociation('field');
$criteria->addAssociation('field.attribute');

/** @var Entity $entity */
$entity = EntityRepositoryInterface $repository->search($criteria, $this->context);
```

### Custom Field in Shopware Plugin
#### Init custom field on plugin install:
```php
public function install(InstallContext $installContext): void
{
	/** @var EntityRepository $customFieldSetRepository */
	$customFieldSetRepository = $this->container->get('custom_field_set.repository');

	$customFieldSetRepository->upsert([
		[
			'name' => '[prefix_custom_field_set]',
			'config' => [
				'label' => [
					'de-DE' => '[LABEL]',
					'en-GB' => '[LABEL]'
				]
			],
			'customFields' => [
				[
					'name' => '[prefix_customfield_name]',
					'type' => CustomFieldTypes::[TYPE],
					'config' => [
						'label' => [
							'de-DE' => '[LABEL]',
							'en-GB' => '[LABEL]'
						],
						'type' => '[TYPE]',
						'customFieldType' => '[TYPE]',
						'customFieldPosition' => 1
					]
				]
			],
			'relations' => [
				[
					'id' => Uuid::randomHex(),
					'entityName' => '[ENTITY_NAME]'
				]
			]
		]
	], $installContext->getContext());
}
```
#### Custom field particularities
##### Text editor as custom field [config values are important, HTML type is not enough]
```php
'name' => '[prefix_customfield_name]',
'type' => CustomFieldTypes::HTML,
'config' => [
	'label' => [
		'de-DE' => '[LABEL]',
		'en-GB' => '[LABEL]'
	],
	'type' => 'HTML',
	'componentName' => 'sw-text-editor',
	'customFieldType' => 'textEditor',
	'customFieldPosition' => 1
]
```
#### Delete custom field on plugin uninstall:
```php
public function uninstall(UninstallContext $uninstallContext): void
{
	$connection = $this->container->get(Connection::class);

	$connection->executeUpdate('
		DELETE FROM `custom_field_set`
		WHERE name LIKE \'[prefix_custom_field_set]\'
	');

	$connection->executeUpdate('
		DELETE FROM `custom_field`
		WHERE name LIKE \'[prefix_customfield_name]\'
	');
}
```
#### Fill custom field in plugin:
```php
public function setCustomField(string $param): void  
{  
	$criteria = new Criteria();  
	$criteria->addFilter(new EqualsAnyFilter('[IDENTIFIER]', [$param]));  
  
	/** @var Entity $entity */  
	$entity = $this->repository->search($criteria, $this->context)->first();  
  
	$this->repository->update([  
		[  
			'id' => $entity->getId(),  
			'customFields' => ['[prefix_customfield_name]' => '[data]']  
		]
	], $this->context);  
}
```

### Page Event Subscription

```php
public static function getSubscribedEvents(): array  
{  
	return [  
		[...Page]LoadedEvent::class => 'on[...]Page'  
	];  
}
```

#### Available Shopware 6 Page Events
#### The Shopware 6 Events can be found in your vendor directory under
```bash
vendor/shopware/storefront/Page/<PAGENAME>LoadedEvent.php
```
