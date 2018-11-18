# React Default Props Pattern with Typescript

I was wondering how to properly type the props and the component when you have default props.
We can find plenty of ways to do it online, so I said to myself: "Sammy, you deserve your own react default props pattern".
More seriously:

- none of them seemed perfect because they involved force typing or HOC
- They were not using the FunctionComponent type of React for arrow functions component

I want something simple that answers to the two following needs:

- When I'm using my component, I want autocompletion on the props of the component
- When I'm coding in my component, I want autocompletion on the props object

My component takes one `mandatoryProp` which is a number, and one `optionalProp` which is either "Yolo", "Croute" or "lol" and will have a default value.

```js
const defaultProps = {
    optionalProp: "croute" as "yolo" | "croute" | "lol"
}

type MandatoryProps = {
    mandatoryProp: number
}
```

Out of this two props I want the type that I will use from the JSX side

```js
type JSXProps = MandatoryProps & Partial<typeof defaultProps>;
```

And the actual props my component will use:

```js
type Props = MandatoryProps & typeof defaultProps;
```

My component will look like this:

```js
// The JSXProps will be used by react to valdiate the use of the component
// The Props will be the actual type after I have
const MyComponent: React.FunctionComponent<JSXProps> = (props: Props) => {
  props = { ...defaultProps, ...props };

  // body of the function

  return (
    <div>
      <p>{prop.mandatoryProp}</p> // which is defined
      <p>{prop.optionalProp}</p> // which is defined
    </div>
  );
};
```

Voila. The complete pattern looks like this:

```js
const defaultProps = {
    optionalProp: "croute" as "yolo" | "croute" | "lol"
}

type MandatoryProps = {
    mandatoryProp: number
}

const MyComponent: React.FunctionComponent<
    MandatoryProps & Partial<typeof defaultProps>
> = (props: MandatoryProps & typeof defaultProps) => {
    props = { ...defaultProps, ...props };

  // body of the function

    return (
        <div>
            <p>{prop.mandatoryProp}</p> // which is defined
            <p>{prop.optionalProp}</p> // which is defined
        </div>
    );
};
```
