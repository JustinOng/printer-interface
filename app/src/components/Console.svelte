<script lang="ts">
  const LINE_BUFFER_SIZE = 256;

  import Card, { Content } from "@smui/card";

  let lines: string[] = [];

  export function addLine(line: string) {
    lines.push(line);

    while (lines.length > LINE_BUFFER_SIZE) {
      lines.shift();
    }

    lines = lines;
  }
</script>

<Card padded>
  <h3 style="margin: 0">Console</h3>
  <Content><pre>{lines.join("\n")}</pre></Content>
</Card>