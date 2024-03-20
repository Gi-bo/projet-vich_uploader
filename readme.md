## Installation de VichUploader
```
composer require vich/uploader-bundle
```

## Modification de `/config/packages/vich_uploader.yaml`

```yaml
mappings:
        posts:
            uri_prefix: /images/posts
            upload_destination: '%kernel.project_dir%/public/images/posts'
            namer: Vich\UploaderBundle\Naming\SmartUniqueNamer

            inject_on_load: false
            delete_on_update: true
            delete_on_remove: true
```

## Ajout de la propriété imageFile dans l'entité `Post`

```php

// à mettre après le "namespace" en haut du fichier
use Symfony\Component\HttpFoundation\File\File;
use Vich\UploaderBundle\Mapping\Annotation as Vich;
use Symfony\Component\Validator\Constraints as Assert;

// à mettre au dessus de la définition de la classe Post
#[Vich\Uploadable]
...

    // NOTE: This is not a mapped field of entity metadata, just a simple property.
    #[Assert\File(maxSize: '4000k',)]
    #[Vich\UploadableField(mapping: 'posts', fileNameProperty: 'imageName', size: 'imageSize')]
    private ?File $imageFile = null;

...

    // vérifier que "string" est précédé d'un "?" dans la fonction
    public function setImageName(?string $imageName): void
    {
        $this->imageName = $imageName;
    }

    /**
     * If manually uploading a file (i.e. not using Symfony Form) ensure an instance
     * of 'UploadedFile' is injected into this setter to trigger the update. If this
     * bundle's configuration parameter 'inject_on_load' is set to 'true' this setter
     * must be able to accept an instance of 'File' as the bundle will inject one here
     * during Doctrine hydration.
     *
     * @param File|\Symfony\Component\HttpFoundation\File\UploadedFile|null $imageFile
     */
    public function setImageFile(?File $imageFile = null): void
    {
        $this->imageFile = $imageFile;

        if (null !== $imageFile) {
            // It is required that at least one field changes if you are using doctrine
            // otherwise the event listeners won't be called and the file is lost
            $this->updatedAt = new \DateTimeImmutable();
        }
    }

    public function getImageFile(): ?File
    {
        return $this->imageFile;
    }

 ...
```

## Modification de `PostType.php`

```php
use Vich\UploaderBundle\Form\Type\VichImageType;

...

    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('title')
            ->add('content')
            ->add('imageFile', VichImageType::class)
        ;
    }

...
```
## Au final : 3 fichiers modifiés

- [`./config/packages/vich_uploader.yaml`](./config/packages/vich_uploader.yaml)
- [`./src/Entity/Post.php`](./src/Entity/Post.php)
- [`./src/Form/PostType.php`](./src/Form/PostType.php)