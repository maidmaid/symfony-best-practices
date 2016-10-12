Uploadable example
==================

```yml
# app/config/config.yml

stof_doctrine_extensions:
    orm:
        default:
            uploadable: true
    uploadable:
        default_file_path: %kernel.root_dir%/../web/uploads
```

```php
// src/AppBundle/Entity/Document.php

use Doctrine\ORM\Mapping as ORM;
use Gedmo\Mapping\Annotation as Gedmo;
use Symfony\Component\Validator\Constraints as Assert;

/**
 * @ORM\Table(name="document")
 * @ORM\Entity(repositoryClass="AppBundle\Entity\DocumentRepository")
 * @Gedmo\Uploadable(filenameGenerator="SHA1", allowOverwrite=true, appendNumber=true)
 */
class Document
{
    /** @ORM\Column(name="id", type="integer") @ORM\Id @ORM\GeneratedValue(strategy="AUTO") */
    private $id;

    /**
     * @var string
     *
     * @ORM\Column(name="file_path", type="string", length=255)
     * @Gedmo\UploadableFilePath
     */
    private $filePath;

    /**
     * @var string
     *
     * @ORM\Column(name="file_name", type="string", length=255)
     * @Gedmo\UploadableFileName
     */
    private $fileName;

    /**
     * @Assert\File()
     */
    public $file;

    // Setters, getters...

    /**
     * Get web path.
     *
     * @return string
     */
    public function getWebPath()
    {
        return $this->fileName ? 'uploads/'.$this->fileName : '';
    }
}
```

```twig
{# app/Resources/views/document/show.html.twig #}
<img src="{{ asset(document.webPath) }}" alt=""/>

{# app/Resources/views/document/edit.html.twig #}
<img src="{{ asset(form.vars.data.webPath) }}" alt=""/>
```

```php
// src/AppBundle/Form/DocumentType.php 

use Symfony\Component\Form\Extension\Core\Type\FileType;

public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder
        ->add('file', FileType::class, array(
            'data_class' => null,
            'required' => false,
        ))
    ;
}
```

```php
// src/AppBundle/Controller/DocumentController.php

public function editAction(Request $request, Document $document)
{
    //...
    if ($editForm->isValid()) {
        $uploadableManager = $this->container->get('stof_doctrine_extensions.uploadable.manager');
            if ($document->file instanceof UploadedFile) {
                $uploadableManager->markEntityToUpload($document, $document->file);
            }
        }
        //...
    }
    //...
```
```php
// src/AppBundle/DataFixtures/ORM/LoadData.php

public function load(ObjectManager $manager)
{
    $uplodableManager = $this->container->get('stof_doctrine_extensions.uploadable.manager');
    //...
    $image = new Document();

    $path = $faker->image(sys_get_temp_dir(), 500, 500, 'nightlife');
    $image->file = new UploadedFile($path, $path);
    $uploadableManager->markEntityToUpload($image, $image->file);

    $article->addImage($image);
    $manager->persist($article);
    //...
}
```

- [Doc uploadable extension](https://github.com/Atlantic18/DoctrineExtensions/blob/master/doc/uploadable.md)
- [Doc stof doctrine extensions](https://github.com/stof/StofDoctrineExtensionsBundle/blob/master/Resources/doc/index.rst)
- [Doc symfony file uploads](http://symfony.com/doc/current/cookbook/doctrine/file_uploads.html)
