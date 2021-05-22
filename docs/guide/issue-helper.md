# Issue Helper

## Before You Start...

The issue list is reserved exclusively for bug reports and feature requests. That means we do not accept usage questions. If you open an issue that does not conform to the requirements, it will be closed immediately.

For usage questions, please use the following resources:

- Read the introduce and components documentation
- Make sure you have search your question in FAQ and changelog
- Look for / ask questions on [Discussions](https://github.com/alibaba/formily/discussions)

Also try to search for your issue 

it may have already been answered or even fixed in the development branch. However, if you find that an old, closed issue still persists in the latest version, you should open a new issue using the form below instead of commenting on the old issue.

```tsx
import React from 'react'
import { createForm, onFieldMount, onFieldReact } from '@formily/core'
import { Field, VoidField } from '@formily/react'
import {
  Form,
  Input,
  Select,
  Radio,
  FormItem,
  FormButtonGroup,
  Submit,
} from '@formily/antd'
import semver from 'semver'
import ReactMde from 'react-mde'
import * as Showdown from 'showdown'
import 'react-mde/lib/styles/css/react-mde-all.css'

const converter = new Showdown.Converter({
  tables: true,
  simplifiedAutoLink: true,
  strikethrough: true,
  tasklists: true,
})

const MdInput = ({ value, onChange, ...props }) => {
  const [selectedTab, setSelectedTab] = React.useState('write')
  return (
    <div style={{ fontSize: 12, lineHeight: 1 }}>
      <ReactMde
        value={value}
        onChange={onChange}
        selectedTab={selectedTab}
        onTabChange={setSelectedTab}
        generateMarkdownPreview={(markdown) =>
          Promise.resolve(
            `<div class="markdown" style="margin:0 20px;">${
              converter.makeHtml(markdown) || ''
            }</div>`
          )
        }
      />
    </div>
  )
}

const form = createForm({
  validateFirst: true,
  effects() {
    onFieldMount('version', async (field) => {
      const { versions: unsort } = await fetch(
        'https://registry.nlark.com/@formily/core'
      ).then((res) => res.json())

      const versions = Object.keys(unsort).sort((v1, v2) =>
        semver.gte(v1, v2) ? -1 : 1
      )
      field.dataSource = versions.map((version) => ({
        label: version,
        value: version,
      }))
    })
    onFieldMount('package', async (field) => {
      const packages = await fetch(
        'https://api.npms.io/v2/search/suggestions?q=@formily'
      ).then((res) => res.json())
      field.dataSource = packages
        .filter(({ package: { name } }) => {
          return name.indexOf('@formily/') > -1
        })
        .map(({ package: { name } }) => {
          return {
            label: name,
            value: name,
          }
        })
    })
    onFieldReact('bug-desc', (field) => {
      field.visible = field.query('type').value() === 'Bug Report'
    })
    onFieldReact('feature-desc', (field) => {
      field.visible = field.query('type').value() === 'Feature Request'
    })
  },
})

export default () => {
  return (
    <Form form={form} layout="vertical" size="large">
      <Field
        title="This is a"
        name="type"
        required
        initialValue="Bug Report"
        decorator={[FormItem]}
        component={[Radio.Group, { optionType: 'button' }]}
        dataSource={[
          { label: 'Bug Report', value: 'Bug Report' },
          { label: 'Feature Request', value: 'Feature Request' },
        ]}
      />
      <Field
        title="Title"
        name="title"
        required
        decorator={[FormItem]}
        component={[Input]}
      />
      <VoidField name="bug-desc">
        <Field
          title="Package"
          name="package"
          required
          decorator={[FormItem]}
          component={[Select, { showSearch: true }]}
        />
        <Field
          title="Version"
          description="Check if the issue is reproducible with the latest stable version."
          name="version"
          required
          decorator={[FormItem]}
          component={[Select, { showSearch: true }]}
        />
        <Field
          title="Link to minimal reproduction"
          name="reproduceLink"
          decorator={[FormItem]}
          component={[Input]}
          required
          validator={[
            'url',
            (value) => {
              return /\/\/codesandbox\.io/.test(value)
                ? ''
                : 'Must Be Codesandbox Link'
            },
          ]}
        />
        <Field
          title="Step to reproduce"
          description="Clear and concise reproduction instructions are important for us to be able to triage your issue in a timely manner. Note that you can use Markdown to format lists and code."
          name="reproduceStep"
          decorator={[FormItem]}
          component={[MdInput]}
          required
        />
        <Field
          title="What is expected?"
          name="expected"
          decorator={[FormItem]}
          component={[MdInput]}
          required
        />
        <Field
          title="What is actually happening?"
          name="actually"
          decorator={[FormItem]}
          component={[MdInput]}
          required
        />
        <Field
          title="Any additional comments? (optional)"
          name="comment"
          decorator={[FormItem]}
          component={[MdInput]}
        />
      </VoidField>
      <VoidField name="feature-desc">
        <Field
          title="What problem does this feature solve?"
          description={
            <div>
              <p>
                Explain your use case, context, and rationale behind this
                feature request. More importantly, what is the end user
                experience you are trying to build that led to the need for this
                feature?
              </p>
              <p>
                An important design goal of Ant Design is keeping the API
                surface small and straightforward. In general, we only consider
                adding new features that solve a problem that cannot be easily
                dealt with using existing APIs (i.e. not just an alternative way
                of doing things that can already be done). The problem should
                also be common enough to justify the addition.
              </p>
            </div>
          }
          name="feature"
          required
          decorator={[FormItem]}
          component={[MdInput]}
        />

        <Field
          title="What does the proposed API look like?"
          description="Describe how you propose to solve the problem and provide code samples of how the API would work once implemented."
          name="api"
          required
          decorator={[FormItem]}
          component={[MdInput]}
        />
      </VoidField>
      <FormButtonGroup.Sticky align="center">
        <Submit size="large">Submit</Submit>
      </FormButtonGroup.Sticky>
    </Form>
  )
}
```