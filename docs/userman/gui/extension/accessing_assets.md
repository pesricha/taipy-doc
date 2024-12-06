# Accessing the library assets

In certain scenarios, you might want to enrich your user interface by displaying a small image alongside text.
For example, when using a caption control to represent a company name along with its logo,
adding an image can enhance visual context and usability.

Traditionally, in HTML, you would use an `img` tag with the `src` attribute pointing to the image’s file path.
However, directly referencing resources in this way can expose your application to potential security vulnerabilities,
such as unauthorized access or malicious resource requests.

To mitigate these risks, Taipy introduces the `ElementLibrary.get_resource()^` method.
This method acts as a secure gateway for resource handling,
allowing the application to validate and filter resource requests based on predefined settings.
It ensures that only authorized and properly configured files are served, protecting your application while maintaining
functionality.

## Declaring element {data-source="gui:doc/extension/example_library/example_library.py#L62"}

In this section, we will create a new element that displays a logo image alongside a text caption. This visual element
will utilize the `(ElementLibrary.)get_resource()^` method to securely access the image file on the server.

```python title="example_library.py"
import base64

from taipy.gui.extension import Element, ElementLibrary, ElementProperty, PropertyType


class ExampleLibrary(ElementLibrary):
    def __init__(self) -> None:
        # Initialize the set of visual elements for this extension library
        logo_path = self.get_resource("assets/logo.png")
        with open(logo_path, "rb") as f:
            logo_base64 = base64.b64encode(f.read()).decode("utf-8")

        self.elements = {
            "logo_with_text": Element(
                "text",
                {
                    "text": ElementProperty(PropertyType.string),
                    "logo_path": ElementProperty(PropertyType.string, default_value=logo_base64),
                },
            )
        }
```

The detailed explanation of the code is as follows:

- The `(ElementLibrary.)get_resource()^`
method retrieves the absolute path to the logo image file on your local file system.
This means the file is not accessible from the front-end.
To make the file accessible via the web, you should use the `(ElementLibrary.)get_resource_url()^` method instead.
- The `logo_with_text` element includes two properties: `text` and `logo_path`.
- The `text` property has the type `PropertyType.string^`, meaning it holds a string value.
- The `logo_path` property has the type `PropertyType.string^`, meaning it holds a string value.
  The `default_value` parameter is set to the base64-encoded image file, which is used as the default value for the property.

## Creating the React component {data-source="gui:doc/extension/example_library/front-end/src/LogoWithText.tsx"}
```tsx title="LogoWithText.tsx"
import React from "react";
import { useDynamicProperty } from "taipy-gui";

interface CaptionProps {
    text: string;
    defaultText: string;
    logoPath: string;
}

const styles = {
    container: {
        display: "flex",
        alignItems: "center",
    },
    logo: {
        width: "4em",
        height: "4em",
        marginRight: "10px",
    },
};

const LogoWithText = ({ text, defaultText, logoPath }: CaptionProps) => {
    const value = useDynamicProperty(text, defaultText, "");

    return (
        <div style={styles.container}>
            <img
                src={`data:image/png;base64,${logoPath}`}
                alt="LogoWithText"
                style={styles.logo}
            />
            <div>{value}</div>
        </div>
    );
};

export default LogoWithText;
```

## Exporting the React component {data-source="gui:doc/extension/example_library/front-end/src/index.ts"}

When the component is entirely defined, it must be exported by the library's JavaScript bundle.
This is done by adding the export directive in the file *<project dir>/<package dir>front-end/src/index.ts*.

```ts title="index.ts"
import LogoWithText from "./LogoWithText";

export { LogoWithText };
```

## Using the element {data-source="gui:doc/extension/logo_with_text.py"}

```python title="logo_with_text.py"
name = "Taipy"

page = """
<|{name}|logo_with_text|>
"""
```

When you run this application, the page displays the element like this:
<figure>
    <img src="../logo_with_text-d.png" class="visible-dark"/>
    <img src="../logo_with_text-l.png" class="visible-light"/>
    <figcaption>Logo with text</figcaption>
</figure>

# Additional resources {data-source="gui:doc/extension/example_library/example_library.py#L92"}

Beside the `ElementLibrary.get_resource()^` method, Taipy provides other methods to manage resources securely.
One of them is the `ElementLibrary.get_scripts()^` method, which allows you to include JavaScript files in your application.
This method ensures that only authorized scripts are loaded, protecting your application from potential security threats.

The code snippet below illustrates how to use the `(ElementLibrary.)get_scripts()^` method to include a JavaScript file that adds animation to your application’s logo.
This snippet should be placed within your extension library class, where you override the `(ElementLibrary.)get_scripts()^` method to return a list of script paths.

```python title="example_library.py"
    def get_scripts(self) -> list[str]:
        return [
            "front-end/dist/exampleLibrary.js",
            "front-end/scripts/logoAnimation.js",
        ]
```

## Custom Javascript file {data-source="gui:doc/extension/example_library/front-end/scripts/logoAnimation.js"}

By adding this custom script to your extension library, it will be included in the application bundle.
When the application is run, the script will be executed, adding animation to the logo image.

```js title="logoAnimation.js"
const style = document.createElement('style');
style.innerHTML = `
@keyframes logoAnimation {
    from {
        transform: scale(1);
    }
    to {
        transform: scale(1.5);
    }
}

.logo-animate {
    animation: logoAnimation 2s infinite alternate;
}
`;
document.head.appendChild(style);

document.addEventListener("DOMContentLoaded", () => {
    const checkForElement = setInterval(() => {
        const logoImage = document.querySelector('img[alt="LogoWithText"]');
        if (logoImage) {
            logoImage.classList.add('logo-animate');
            clearInterval(checkForElement);
        }
    }, 100);
});
```

<figure>
    <img src="../animation_logo_with_text-l.gif" class="visible-light" alt="Logo with Animation">
    <img src="../animation_logo_with_text-d.gif" class="visible-dark" alt="Logo with Animation">
    <figcaption>Logo with Animation</figcaption>
</figure>

!!! info
    Keep in mind that there will be a delay between the page loading and the component rendering.
    The script will execute only after the page has fully loaded and the component has been rendered.


